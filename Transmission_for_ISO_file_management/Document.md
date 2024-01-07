# Arch Linux ServerにTransmissionを入れ，LinuxOSインストールディスクを管理しやすくする

## 目的
実機，Proxmox共にOSを入れ替えるケースが多い
Transmissionを導入し，torrentを用いてISOファイルをダウンロードし，これをLAN内向けのNGINXサーバで公開する
権限回りやディレクトリ設計はあまりよくないかもしれない

## Arch Linuxのインストール等
割愛
既に固定IPなどもの設定も完了しているものとする

## Transmissionのインストール

rootユーザにて

```bash
# pacman -S transmission-cli
```

## Transmission用のユーザを追加
```bash
# useradd -m -g users -G transmission -s /bin/bash tr-user
# passwd tr-user
```

## 一度Transmission用ユーザにてTransmissionを実行
```bash
# su tr-user
$ transmission-daemon
$ killall transmission-daemon
```

## Systemdでの実行ユーザを設定
```bash
# mkdir /etc/systemd/system/transmission.service.d
# echo '[Service]' > /etc/systemd/system/transmission.service.d/username.conf
# echo 'User=tr-user' >> /etc/systemd/system/transmission.service.d/username.conf
```

## Web UIにアクセスするIPアドレスを追加
Transmission用ユーザにて
```bash
$ vim ~/.config/transmission-daemon/settings.json
```
ファイル内の"rpc-whitelist"項目に許可するIPアドレスリストを追加する
ワイルドカードは"*"

## Transmissionの自動起動設定
```bash
# systemctl enable transission
```

## nginxのインストール
```bash
# pacman -S nginx
```

## Transmissionのダウンロードディレクトリの権限設定
Transmissionユーザにて
```bash
$ cd ~
$ chmod 755 ./Downloads
$ cd ~/Downloads
$ chmod 644 *
$ cd /home
$ chmod 701 tr-user
```

## nginxの設定
```bash
# vim /etc/nginx/nginx.conf
```
下記の2行をserver内に追記
```
autoindex on;
charset utf-8;
```
該当箇所を下記のように修正
```
location / {
    root    /home/tr-user/Downloads;
}
```

## nginxの自動起動設定
```bash
# systemctl enable nginx
```

## サーバ再起動
```bash
# reboot
```
