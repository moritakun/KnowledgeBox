# 目次

1. S3連携構成
2. IAM連携構成
3. CloudWatch連携構成
4. VPC/ネットワーク構成
5. まとめ

---

## 1. S3連携構成

### 1.1 概要

DatabricksクラスターからAWS S3へのデータアクセスは、Instance Profileを通じたIAM認証により実現される。<br>
DBFSマウントポイントを使用することで、S3バケットをローカルファイルシステムのように扱うことが可能。

### 1.2 主要構成要素

| 構成要素 | 役割 | 設定項目 |
|----------|------|----------|
| Instance Profile | DatabricksクラスターにIAMロールを付与し、S3へのアクセス権限を提供 | IAMロールARN、信頼ポリシー、S3アクセスポリシー |
| S3 Bucket | データレイク、Raw/Processed/Archiveの各層でデータを管理 | バケット名、暗号化設定、ライフサイクルポリシー、バージョニング |
| DBFS Mount | S3バケットをDatabricksファイルシステムにマウント | マウントポイント名、S3パス、認証情報設定 |
| S3 Bucket Policy | バケットレベルでのアクセス制御、特定のIAMロールからのアクセスを許可 | Principal設定、Action許可、Resource指定 |

### 1.3 接続方式

- **直接アクセス方式**: spark.read/write APIでS3パスを直接指定
- **DBFSマウント方式**: S3バケットをDBFSパスとしてマウント
- **Unity Catalogテーブル**: メタデータ管理とアクセス制御を統合

### 1.4 実装例

DBFSマウント設定のPythonコード例：

```python
# S3バケットをDBFSにマウント
dbutils.fs.mount(
    source = "s3a://my-bucket/raw-data",
    mount_point = "/mnt/raw-data",
    extra_configs = {
        "fs.s3a.aws.credentials.provider": 
        "com.amazonaws.auth.InstanceProfileCredentialsProvider"
    }
)

# マウント済みパスからデータ読み込み
df = spark.read.parquet("/mnt/raw-data/2025/01/")
```

---

## 2. IAM連携構成

### 2.1 概要

DatabricksとAWS間の認証・認可は、Cross-Account IAMロールとInstance Profileの組み合わせにより実現される。<br>
これにより、Databricksクラスターが安全にAWSリソースへアクセスできる。

### 2.2 主要構成要素

| 構成要素 | 役割 | 設定項目 |
|----------|------|----------|
| Cross-Account IAM Role | DatabricksアカウントからのAssumeRoleを許可するロール | Trust Policy (DatabricksアカウントID)、External ID、ロール名 |
| Instance Profile | EC2インスタンス（クラスターノード）に権限を付与 | 関連付けるIAMロール、Instance Profile ARN |
| Trust Policy | どのプリンシパルがロールを引き受けられるかを定義 | Principal (AWS/Service)、Condition (sts:ExternalId) |
| Permission Policy | ロールに付与される実際のアクセス権限 | Action (s3:*, kms:*, secretsmanager:*)、Resource ARN |
| SCIM/SSO統合 | ユーザー認証とID管理の自動化 | IdPメタデータ、SCIM Token、属性マッピング |

### 2.3 IAMポリシー設計

- **最小権限の原則**: 必要最小限の権限のみを付与
- **条件付きアクセス**: IPアドレス、VPC、タグベースの制限
- **リソースベース制約**: 特定のS3バケットやKMSキーへのアクセスに限定
- **定期的な権限監査**: AWS Access Analyzerを使用した不要な権限の検出

### 2.4 信頼Policyの例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/databricks-role"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "databricks-external-id-12345"
        }
      }
    }
  ]
}
```

---

## 3. CloudWatch連携構成

### 3.1 概要

DatabricksクラスターのログとメトリクスをAWS CloudWatchに転送することで、統合的な監視・アラート・ログ分析が可能。<br>
クラスターログ、ジョブログ、アプリケーションログを一元管理可能。

### 3.2 主要構成要素

| 構成要素 | 役割 | 設定項目 |
|----------|------|----------|
| Cluster Log Delivery | クラスターログをS3経由でCloudWatch Logsに転送 | S3バケットパス、CloudWatch Logs Group、ストリーム名 |
| CloudWatch Logs | ログの集約、検索、フィルタリング、長期保存 | ロググループ名、保持期間、サブスクリプションフィルター |
| CloudWatch Metrics | クラスターメトリクス（CPU、メモリ、ディスク）の収集と可視化 | カスタムメトリクス、名前空間、ディメンション |
| CloudWatch Alarms | メトリクスの閾値監視と自動アラート | 閾値設定、評価期間、アクション（SNS通知） |
| CloudWatch Dashboards | メトリクスとログの統合ダッシュボード | ウィジェット構成、時系列グラフ、ログインサイト |
| SNS Topic | アラーム通知の配信先 | サブスクリプション（Email、Lambda、SQS） |

### 3.3 ログの種類

- **クラスターログ**: Driver/Workerノードのシステムログ、初期化スクリプトログ
- **ジョブログ**: Sparkジョブの実行ログ、エラー、警告
- **アプリケーションログ**: ユーザーコードからの出力、カスタムロギング
- **監査ログ**: ユーザーアクション、アクセス履歴（Workspace Audit Logs）

### 3.4 クラスターログ配信設定

```json
# Cluster configuration (JSON)
{
  "cluster_name": "my-analytics-cluster",
  "cluster_log_conf": {
    "s3": {
      "destination": "s3://my-logs-bucket/databricks-logs/",
      "region": "ap-northeast-1",
      "enable_encryption": true
    }
  }
}
```

```bash
# CloudWatch Logs Subscription Filter (AWS CLI)
aws logs put-subscription-filter \
  --log-group-name /databricks/cluster-logs \
  --filter-name spark-errors \
  --filter-pattern "[ERROR]" \
  --destination-arn arn:aws:lambda:ap-northeast-1:123456789012:function:log-processor
```

---

## 4. VPC/ネットワーク構成

### 4.1 概要

DatabricksクラスターはAWSのVPC内にデプロイされ、セキュアなネットワーク構成により外部との通信を制御している。<br>
VPCエンドポイントを使用することで、AWSサービスへのプライベート接続が可能。

### 4.2 主要構成要素

| 構成要素 | 役割 | 設定項目 |
|----------|------|----------|
| VPC（Virtual Private Cloud） | Databricksクラスター専用の隔離されたネットワーク環境 | CIDR範囲、VPC ID、リージョン |
| プライベートサブネット | クラスターノード（Driver/Worker）が配置される非公開サブネット | サブネットCIDR、可用性ゾーン、ルートテーブル |
| パブリックサブネット | NAT Gateway配置用、外部通信の出口 | サブネットCIDR、Internet Gateway関連付け |
| Security Group | クラスター間通信の制御、インバウンド/アウトバウンドルール | インバウンドルール（内部通信）、アウトバウンドルール（AWS API等） |
| NAT Gateway | プライベートサブネットからの外部通信を可能にする | Elastic IP、配置サブネット |
| VPC Endpoint | AWSサービス（S3、STS、KMS）へのプライベート接続 | エンドポイントタイプ（Gateway/Interface）、サービス名、ポリシー |
| Network ACL | サブネットレベルのステートレスファイアウォール | インバウンドルール、アウトバウンドルール、ルール番号 |

### 4.3 ネットワーク設計パターン

- **フルプライベート構成**: VPCエンドポイント使用、インターネット接続なし
- **ハイブリッド構成**: プライベート接続とNAT Gateway併用
- **マルチAZ構成**: 高可用性のための複数可用性ゾーン配置
- **PrivateLink統合**: Databricks Control Planeへのプライベート接続

### 4.4 VPC Endpoint設定例

```bash
# S3 Gateway Endpoint (AWS CLI)
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.ap-northeast-1.s3 \
  --route-table-ids rtb-11111111 rtb-22222222

# STS Interface Endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.ap-northeast-1.sts \
  --subnet-ids subnet-aaaa1111 subnet-bbbb2222 \
  --security-group-ids sg-12345678
```

---

## 5. まとめ

### 5.1 調査内容の総括

| 連携項目 | 主要技術 | セキュリティ考慮点 | 運用ポイント | 推奨設定 |
|----------|----------|-------------------|-------------|----------|
| S3連携 | Instance Profile, DBFS Mount | 暗号化、バケットポリシー | ライフサイクル管理、コスト最適化 | データレイク3層構造 |
| IAM連携 | Cross-Account Role, Trust Policy | 最小権限、条件付きアクセス | 定期的な権限監査 | External ID使用 |
| CloudWatch連携 | Logs, Metrics, Alarms | ログ暗号化、アクセス制御 | アラート設定、ダッシュボード | ログ保持期間の設定 |
| VPC/ネットワーク | VPC Endpoint, Security Group | プライベート接続、NACLs | マルチAZ、冗長性 | PrivateLink使用 |

### 5.2 今後の学習課題

- **Unity Catalogとの統合**: データガバナンスとアクセス制御の高度化
- **Delta Lake最適化**: Z-orderingとデータスキッピングの活用
- **コスト最適化**: Spot Instanceの活用とクラスター自動スケーリング
- **CI/CD統合**: Databricks Asset Bundlesによる自動デプロイ
- **MLOps実践**: MLflowとモデルレジストリの活用

### 5.3 参考リソース

- Databricks公式ドキュメント: https://docs.databricks.com/