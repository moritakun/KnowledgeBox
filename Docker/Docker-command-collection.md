# Dockerコマンド集

# カスタマイズしたコンテナの保存

1. コンテナイメージを確認する
`docker images`
2. コンテナイメージの作成
`docker commit <コンテナ名 or ID> <イメージ名>:<タグ名>`
※イメージ名は、小文字である必要があります
※コンテナを起動した状態でイメージ作成する場合は、`—pause=false` オプションをつける（無くても行けた）
3. イメージが作成されたか確認
`docker images` 

# コンテナ名の変更

`docker rename <old_contena_name> <new_contena_name>` 

# dockerコマンドが実行できなくなった場合

1. docker updateがあるかを確認し、updateする
2. dockerを再起動する

# コンテナ削除

`docker rm <コンテナ名>`
※削除対象のコンテナは停止している必要がある

# イメージ削除

`docker rmi <イメージ名>`
※削除対象のイメージがコンテナで利用されている場合は削除できない

# その他

- dockerコンテナ内では、systemdは利用できない
→どうにかこうにか頑張ればできるようだが、かなりめんどくさそう。。
→コンテナ内でプロセス管理をする場合は、「**Supervisor**」を利用するのが一般的ぽい
    - プロセスの考え方については、[こちら](https://qiita.com/hirotaka-tajiri/items/d350a9f1f99779c59fd4)が参考になる