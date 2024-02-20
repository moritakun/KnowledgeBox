# Jenkins

タグ: CI/CD, Jenkins
作成日時: 2024年2月13日 12:39
最終更新日時: 2024年2月13日 13:00

# 概要

Jenkins(ジェンキンス) とは、継続的インテグレーション(CI)や継続的デリバリー(CD)、継続的デプロイメントを実現するツールです。 Jenkinsは、「ソフトウェアのリリーススピードの向上」「開発プロセスの自動化」「開発コストの削減」といった目的とするオープンソースのツールです。

以下の流れをjenkinsパイプラインで繋いで、CIを行っている。

1. ソースコードリポジトリ（github, gitlab）
2. アーティファクトリポジトリ（SonaType Nexus, JFrogArtifactory）
3. CI/CDツール（jenkins,CircleCI,CodePipeline）
4. ビルドツール（maven,Gradle）
5. テストツール（Findbugs(Spotbugs),CheckStyle,JUnit,Selenium,JMeter）
6. デプロイツール（jenkinsのパイプライン）

---

# Appendix

[公式サイト](https://www.jenkins.io/)