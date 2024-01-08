# ssh，sshdに関する設定手順メモ
パスワードログインが可能な状態から設定

## 端末側での鍵生成
```bash
$ ssh-keygen -t ed25519
```

既に生成済みの秘密鍵のパスワードを変更したい場合
```bash
$ ssh-keygen -p -f ~/.ssh/id_ed25519
```

## サーバに公開鍵を設定
```bash
$ ssh-copy-id -i ~/.ssh/id_ed25519 username@serverIP
```

## サーバ側でのセキュリティ向上

すべて設定は"/etc/ssh/sshd\_config"内に記述する．

### パスワードによるログインを禁止
```
PasswordAuthentication no
```

### rootユーザでのログインを禁止
```
PermitRootLogin no
```

### 使用ポートの変更（※例）
```
Port 51324
```

設定後はsshd.serviceを再起動する必要あり
```bash
$ sudo systemctl restart sshd
```

## サーバ側のGUIアプリケーションを端末で使用する場合
サーバ側の"/etc/ssh/sshd\_config"にて下記を設定
```
X11Forwarding yes
```

## クライアント側でのconfigファイルの作成
サーバのIPアドレス，ポート番号，ユーザ名等を毎回打つのは面倒くさい．
また，途中少し調べ事などをしていて接続している端末を離れ，しばらくして戻ると接続が切れている等もある．
さらにはX11のフォワーディング等も絡むと余計面倒くさい．
この辺を楽にするために"~/.ssh/config"を記述する．下記は例である．

```
# GitHubについてはとりあえずこれを書いておく
Host github github.com
        HostName github.com
        IdentityFile ~/.ssh/id_ed25519
        User git

# サーバの例
Host nginx-server                       #sshコマンドを打つ際に使用する文字列
        HostName 192.168.1.100          #サーバのIPアドレス，あるいはURL
        Port 51324                      #使用するポート番号
        IdentityFile ~/.ssh/id_ed25519  #公開鍵認証の場合の秘密鍵の指定
        User user-rt                    #サーバ側でのユーザ名
        ServerAliveInterval 60          #指定秒数毎に生きていることのメッセージを飛ばす
        ServerAliveCountMax 3           #上記にてサーバが応答しなかった場合のリトライ回数
        Compression yes                 #データ圧縮を行うかどうか（yes/no）
        CompressionLevel 6              #Conpressionがyesの場合設定意義あり．1（高速）～9（高圧縮率）の値をとる．
        ForwardX11 yes                  #X11転送を有効にする．-Xと同義
        ForwardX11Trusted yes           #X11転送が有効の場合に設定．ForwardX11とこれの両者がyesの場合には-Yと同義．
```
X11に関してはサーバ側でX11Forwardingが無効の場合，いくらクライアントにて上記設定をしたところで無意味．
