# コンテナにSSH接続をする

# 参考

[YouTube](https://youtu.be/GicWz2OF0sk?si=-qTURlTURUeH5cka)

# コンテナの作成接続

1. 適当なイメージからコンテナを作成
2. コンテナに接続

# sshサーバのインストール

1. `openssh-server` のインストール<br>
`apt install openssh-server`
2. サービスの状態確認<br>
`service --status-all`<br>
※[-] ssh は、サービスが停止状態であること[+]は起動状態 
3. rootユーザのPW設定<br>
`passwd root`
4. sshデーモン（sshd）の設定
    1. 設定ファイルの作成<be>
    `touch /etc/ssh/sshd_config.d/sshd_ubuntu.conf`
    2. `sshd_ubuntu.conf` の編集<br>
    `vi sshd_ubuntu.conf`

        ```powershell
        Port 22
        PasswordAuthentication yes
        PermitRootLogin yes
        ```

5. sshデーモン（sshd）の再起動<br>
`service ssh start`<br>
※Starting OpenBSD Secure Shell server sshdが表示されたこと
6. サービスの状態確認<br>
`service --status-all`<br>
※[-] ssh は、サービスが停止状態であること[+]は起動状態
7. ターミナルでssh接続を確認<br>
`ssh [root@localhost](mailto:root@localhost) -p <port番号>`<br>
rootPWを求められるので、手順3で設定したPWを入力する
8. ログインできたこと