# ubuntuコンテナをGUI表示する方法 <!-- omit in toc -->
- [GUIで表示できるようにする](#guiで表示できるようにする)
  - [ubuntu-desktopのインストール](#ubuntu-desktopのインストール)
  - [GNOMEインストール](#gnomeインストール)
  - [TigerVNCのインストール](#tigervncのインストール)
  - [noVNTのインストール](#novntのインストール)
  - [websockifyのインストール](#websockifyのインストール)
  - [ubuntu-desktopのインストール](#ubuntu-desktopのインストール-1)
  - [GNOMEインストール](#gnomeインストール-1)
  - [TigerVNCのインストール](#tigervncのインストール-1)
  - [noVNTのインストール](#novntのインストール-1)
  - [websockifyのインストール](#websockifyのインストール-1)

## GUIで表示できるようにする

### ubuntu-desktopのインストール

1. `ubuntu-desktop`をインストール
`apt install ubuntu-desktop`
※インストールに軽く30minくらいかかった
2. OS再起動
exitでコンテナから抜けて、`docker restart <コンテナ名 or ID>`を実行

### GNOMEインストール

1. gnomeのインストール
`apt install gnome`

### TigerVNCのインストール

1. TigerVNCをインストール
`apt install tigervnc-standalone-server tigervnc-common tigervnc-tools`  
2. TigerVNC起動
`vncserver`
※デスクトップにアクセスするパスワードを設定する
3. 基本設定
`touch ~/.vnc/xstartup`
    1. xstartup編集
        
        ```powershell
        #!/bin/bash
        /etc/X11/Xtigervnc-session
        gnome-session
        ```
        
    2. ファイル権限変更
    `chmod +x ./vnc/xstartup`
4. config設定
`touch ~/.vnc/config`
    1. config編集
        
        ```powershell
        session=ubuntu
        geometry=1200x720
        alwaysshared
        ```
        
        ※`session=ubuntu`としているのは起動するセッション名`/usr/share/xsessions/`内の`.desktop`ファイルの名前を指定する必要があるため
        
    2. ファイル権限変更
    `chmod +x ./vnc/config` 
5. TigerVNCへアクセスできるユーザを設定
    1. `/etc/tigervnc/vncserver.users`を編集
        
        ```powershell
        # TigerVNC User assignment
        #
        # This file assigns users to specific VNC display numbers.
        # The syntax is <display>=<username>. E.g.:
        #
        # :2=andrew
        # :3=lisa
        :2=<ユーザ名>
        ※システムユーザである必要がある？
        ```
        
6. サービスファイルのコピー
`cp /usr/lib/systemd/system/tigervncserver\@.service /etc/systemd/system/tigervncserver\@.service`
    1. サービスファイルの編集
    `ExecStart=/usr/libexec/tigervncsession-start %i` を`ExecStart=/usr/libexec/tigervncsession-start　<ユーザ名> %i` に修正
    ※`ExecStart=/usr/libexec/tigervncsession-start　<ユーザ名> %i`は　`/etc/tigervnc/vncserver.users` の記載ユーザと念の為同一にする
7. 設定ファイルで定義したユーザのパスワードを設定する
`su - <ユーザ>`
`vncpasswd`
8. TigerVNC起動
`vncserver`
9. 

### noVNTのインストール

1. noVNCのインストール
`apt install novnc`

### websockifyのインストール

1. websockifyのインストール
`apt install websockify`## GUIで表示できるようにする

### ubuntu-desktopのインストール

1. `ubuntu-desktop`をインストール
`apt install ubuntu-desktop`
※インストールに軽く30minくらいかかった
2. OS再起動
exitでコンテナから抜けて、`docker restart <コンテナ名 or ID>`を実行

### GNOMEインストール

1. gnomeのインストール
`apt install gnome`

### TigerVNCのインストール

1. TigerVNCをインストール
`apt install tigervnc-standalone-server tigervnc-common tigervnc-tools`  
2. TigerVNC起動
`vncserver`
※デスクトップにアクセスするパスワードを設定する
3. 基本設定
`touch ~/.vnc/xstartup`
    1. xstartup編集
        
        ```powershell
        #!/bin/bash
        /etc/X11/Xtigervnc-session
        gnome-session
        ```
        
    2. ファイル権限変更
    `chmod +x ./vnc/xstartup`
4. config設定
`touch ~/.vnc/config`
    1. config編集
        
        ```powershell
        session=ubuntu
        geometry=1200x720
        alwaysshared
        ```
        
        ※`session=ubuntu`としているのは起動するセッション名`/usr/share/xsessions/`内の`.desktop`ファイルの名前を指定する必要があるため
        
    2. ファイル権限変更
    `chmod +x ./vnc/config` 
5. TigerVNCへアクセスできるユーザを設定
    1. `/etc/tigervnc/vncserver.users`を編集
        
        ```powershell
        # TigerVNC User assignment
        #
        # This file assigns users to specific VNC display numbers.
        # The syntax is <display>=<username>. E.g.:
        #
        # :2=andrew
        # :3=lisa
        :2=<ユーザ名>
        ※システムユーザである必要がある？
        ```
        
6. サービスファイルのコピー
`cp /usr/lib/systemd/system/tigervncserver\@.service /etc/systemd/system/tigervncserver\@.service`
    1. サービスファイルの編集
    `ExecStart=/usr/libexec/tigervncsession-start %i` を`ExecStart=/usr/libexec/tigervncsession-start　<ユーザ名> %i` に修正
    ※`ExecStart=/usr/libexec/tigervncsession-start　<ユーザ名> %i`は　`/etc/tigervnc/vncserver.users` の記載ユーザと念の為同一にする
7. 設定ファイルで定義したユーザのパスワードを設定する
`su - <ユーザ>`
`vncpasswd`
8. TigerVNC起動
`vncserver`
9. 

### noVNTのインストール

1. noVNCのインストール
`apt install novnc`

### websockifyのインストール

1. websockifyのインストール
`apt install websockify`