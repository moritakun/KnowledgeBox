# コンテナにSSH接続をする

# 参考

[YouTube](https://youtu.be/GicWz2OF0sk?si=-qTURlTURUeH5cka)

# コンテナの作成接続

1. 適当なイメージからコンテナを作成
2. コンテナに接続

# sshサーバのインストール

1. `openssh-server` のインストール
`apt install openssh-server`
2. sshデーモン（sshd）の設定
    1. 設定ファイルの作成
    `touch /etc/ssh/sshd_config.d/sshd_ubuntu.conf`
    2. `sshd_ubuntu.conf` の編集
    `vi sshd_ubuntu.conf` 
        
        ```powershell
        Port 22
        PasswordAuthentication yes
        ```
        
3. sshデーモン（sshd）の再起動
`systemctl restart ssh`