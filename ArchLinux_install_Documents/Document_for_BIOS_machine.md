# BIOSマシンへのArch Linuxのインストール手順
## 実施環境
Proxmox VE 8.1.3上にArch LinuxのVMを建てる時の手順
実マシンでも同様
UEFIマシンではパーティションフォーマットやブートローダに関する手順が違う場合あり．
Live Disk ISOはDownload済み，これを用いた起動も行った後からの作業手順である．
arch-chroot前のプロンプトを1行目，後のプロンプトを2行目，再起動後のrootユーザでのログイン時のプロンプトを3行目のように示す．
```bash
[live-disk]#
[arch-chroot]#
[root]#
```
## システムクロック更新
```bash
[live-disk]# timedatectl set-ntp true
```
正しい時間かどうかを下記で確認
```bash
[live-disk]# timedatectl status
```

## パーティション分け
今回のインストール先デバイスを"/dev/sda"とする．
partedによりパーティション分けを行う
今回は/dev/sda2をルートディレクトリ，/dev/sda1を/bootディレクトリにする
partedを実行するとpartedの対話待機状態になる
```bash
[live-disk]# parted /dev/sda
(parted) mklabel msdos
(parted) mkpart primary ext4 1MiB 512MiB
(parted) mkpart primary ext4 512MiB 100%
(parted) p
(parted) q
```

## ファイルシステムの作成
作成したパーティションをフォーマットする
```bash
[live-disk]# mkfs.ext4 /dev/sda1
[live-disk]# mkfs.ext4 /dev/sda2
```

## マウント
作成したデバイスをマウントしていく
```bash
[live-disk]# mount /dev/sda2 /mnt
[live-disk]# mkdir /mnt/boot
[live-disk]# mount /dev/sda1 /mnt/boot
```

## ベースシステム，その他のインストール
ベースシステムやその他内部で使うものをインストールする
```bash
[live-disk]# pacstrap /mnt base base-devel linux linux-firmware os-prober grub vim sudo
```

## fstabの作成
```bash
[live-disk]# genfstab -U /mnt >> /mnt/etc/fstab
```

## chroot
chrootしてインストール先に入り，これからの作業を行っていく
```bash
[live-disk]# arch-chroot /mnt /bin/bash
```
プロンプトが変わっていることを確認

## ロケールの設定
```bash
[arch-chroot]# echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
[arch-chroot]# echo "ja_JP.UTF-8 UTF-8" >> /etc/locale.gen
[arch-chroot]# locale-gen
```

## タイムゾーンの設定
```bash
[arch-chroot]# ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
[arch-chroot]# hwclock --systohc --utc
```

## ホストネームの設定
例として今回ホストネームを"my-arch"とする．適宜読み替えること．

/etc/hostnameについて
```bash
[arch-chroot]# echo my-arch > /etc/hostname
```

/etc/hostsについて
```bash
[arch-chroot]# vim /etc/hosts
```
下記のように記述
```
127.0.0.1 localhost.localdomain localhost
::1 localhost.localdomain localhost
127.0.1.1 my-arch.localdomain my-arch
```

## rootパスワードの設定
適宜好きなパスワードを設定
```bash
[arch-chroot]# passwd
```

## ユーザの追加
sudoを使えるユーザを作成する．ユーザ名は"rintaro-t"とするので適宜読み替えること．
```bash
[arch-chroot]# useradd -m -g users -G wheel -s /bin/bash rintaro-t
[arch-chroot]# passwd rintaro-t
```

## 作成したユーザにてsudoを使えるようにする
```bash
[arch-chroot]# visudo
```

下記を追加．ユーザ名が"rintaro-t"の場合
```
rintaro-t ALL=(ALL:ALL) ALL
```

※visudo実行時に下記のように怒られた場合はvimのシンボリックリンクをviとして作成
```bash
[arch-chroot]# visudo
visudo: no editor found (editor path = /usr/bin/vi)
[arch-chroot]# ln -sf /usr/bin/vim /usr/bin/vi
```

## GRUBのインストール
今回はブートローダとしてGRUBを使う
```bash
[arch-chroot]# grub-install --target=i386-pc --recheck /dev/sda
[arch-chroot]# grub-mkconfig -o /boot/grub/grub.cfg
```

## systemd-networkdの設定
今回は有線アダプタでDHCPを用いる．これはルータ側のDHCP払い出しにて固定的なIPを払い出す設定を行っているため．
固定IP等を直接書きたい場合等は[こちら](https://wiki.archlinux.jp/index.php/Systemd-networkd)を参照．
まずはネットワークデバイス名を調べる
```bash
[arch-chroot]# ip addr
```
"enp\*"や"ens\*"，"eno\*"，"eth\*"のように表記される．今回はens18であった前提で記す．適宜読み替えること．

```bash
[arch-chroot]# vim /etc/systemd/network/20-wired.network
```
下記のように記述
```
[Match]
Name=ens18

[Network]
DHCP=yes
```

systemd-networkdが自動起動するように設定
```bash
[arch-chroot]# systemctl enable systemd-networkd
```

## 再起動
```bash
[arch-chroot]# exit
[live-disk]# umount -R /mnt
[live-disk]# reboot
```
※ここで起動順位がLiveDiskのほうが上の場合再起動時にもLiveDiskが立ち上がってしまうため，一旦シャットダウンし，LiveDiskを排除する．

## systemd-resolvedの設定
再起動後，rootユーザでログインして作業
```bash
[root]# ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
[root]# systemctl enable systemd-resolved
```
## 再起動
```bash
[root]# reboot
```

以上
