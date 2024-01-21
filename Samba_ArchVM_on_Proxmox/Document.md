# Proxmox VE 8.1.3上のArch VMに実HDDをマウントしてSambaサーバのストレージとして使う
やること
 - Proxmox VEを入れているサーバの残りのストレージをArch VM上から利用する
 - そのArch VM上に該当ストレージを個人で使うようなSambaサービスを構築する
 - また，一時的にファイルを共有する等の用途のためにRAMディスクをSambaの共有ディレクトリ配下に置く

Proxmox VEのルートユーザのプロンプトを下記1行目，Arch VM内のルートユーザのプロンプトを2行目，Arch VM内で使用する一般ユーザのプロンプトを3行目のように表すものとする．
```bash
[pve]# 
[arch]# 
[user1]$
```
## サーバのHDDにProxmox上のVMからアクセスできるようにする
### HDDのIDを調べる
```bash
[pve]# ls /dev/disk/by-id|grep scsi
```
パーティション分け等からとりあえずProxmox VEがインストールされているディスクとそれ以外のディスクを見分ける必要がある．
ここで今回VMからアクセスしたいidを"scsi-36aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"とする

### HDDをマウントできるようにVMのコンフィグファイルに書き込む
```bash
[pve]# qm set 105 -scsi1 /dev/disk/by-di/scsi-36aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```
ここで105がVMの番号，-scsi1はscsi1として認識させる旨を示す．
複数のHDDをマウントする場合，scsi番号を被らないように設定する必要あり．

設定ファイルに書き込まれたかは下記コマンドより確認可能
```bash
[pve]# cat /etc/pve/nodes/server-name/qemu-server/105.conf
```
ここで"server-name"はProxmox VE自体のホストネームになる．適宜書き換えのこと．
また，105.confはVMの番号に応じて書き換えの必要あり．

## Arch VM上にてHDDのマウント関係
まずVMを起動する．すでに起動していた場合は一旦VMをシャットダウンし，再度起動する．
### デバイスを調べる
```bash
[user1]$ ls /dev
```
今回新規追加されたHDDデバイスを/dev/sdbとする．
### パーティションの構築，フォーマット
必要なら行う．必要なければ行わない．
partedコマンドの使い方の詳細は書かない．
```bash
[user1]$ sudo pacman -S parted
[user1]$ sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary ext4 1MiB 100%
(parted) p
(parted) q
[user1]$ sudo mkfs.ext4 /dev/sdb1
```
### マウント
今回のマウント先は一般ユーザの"~/mnt/scsi1/"とする．
```bash
[user1]$ mkdir ~/mnt
[user1]$ mkdir ~/mnt/scsi1
[user1]$ sudo mount /dev/sdb1 ~/mnt/scsi1
```
### 自動マウント設定
起動時に自動的にマウントするようにする．
```bash
[user1]$ sudo pacman -S arch-install-scripts
[user1]$ su
[arch]# genfstab -U /|grep -A 2 /dev/sdb1 >> /etc/fstab
```

## RAMディスクのマウント
### 事前準備
user1のuid及びgidを調べる
```bash
[user1]$ id
```
今回はuid=1000，gid=1000であるものとする．
### マウント
マウントポイントは"~/mnt/ramdisk"とする．
また，当VMにはかなり大きなRAMを割り当てており，今回約20 GBのRAMディスクをマウントするものとする．
正確にはMiBとMBの違いだのなんだのあるが面倒なので適当．
```bash
[user1]$ mkdir ~/mnt/ramdisk
[user1]$ sudo mount -t tmpfs -o uid=1000,gid=1000,size=20000M tmpfs ~/mnt/ramdisk
```
### 自動マウント設定
/etc/fstabに追記する．
```bash
[arch]# echo 'tmpfs /home/user1/mnt/ramdisk tmpfs rw,size=20G,noexec,nodev,nosuid,uid=1000,gid=1000,mode=1700 0 0' >> /etc/fstab
```
## Arch VMにSamba環境を構築する
インストール
```bash
[user1]$ sudo pacman -S samba
```
設定
```bash
[user1]$ sudo vim /etc/samba/smb.conf
```
```
[global]
  deadtime = 60
  disable netbios = yes
  dns proxy = no
  hosts allow = 10.0.5.
  invalid users = root
  security = user
  map to guest = Bad User
  max connections = 100
  workgroup = WORKGROUP
  log file = /path/to/logs/dir/logs.log
  inherit owner = yes
  create mask = 0664
  directory mask = 0775
  force create mode = 0664
  force directory mode = 0775

[user1]
  case sensitive = yes
  valid users = user1
  max log size = 100
  server min protocol = SMB2
  server role = standalone server
  server string = Samba File Server
  use sendfile = yes
  workgroup = WORKGROUP
  path = /home/user1/mnt
  security = user
  writable = yes
```

ユーザ追加
```bash
[arch]# pdbedit -a -u user1
```

サービスの自動起動および起動
```bash
[arch]# systemctl enable smb nmb winbind
[arch]# systemctl start smb nmb winbind
```
