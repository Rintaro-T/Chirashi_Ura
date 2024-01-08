# WSL2上にArch Linux環境を構築する
※作業環境はUbuntu on WSL2

## bootstrapのDownload
[こちら](https://archlinux.org/download/)から好きなミラーを選択し，bootstrapをDownloadする．
今回は[こちら](https://ftp.jaist.ac.jp/pub/Linux/ArchLinux/iso/2024.01.01/)を選択．
WSL2上で作業用ディレクトリを作成しwgetでDownload，一応sha256sumも確認．
```bash
$ mkdir /mnt/c/Users/windows-username/Downloads/tmp
$ cd /mnt/c/Users/windows-username/Downloads/tmp
$ wget https://ftp.jaist.ac.jp/pub/Linux/ArchLinux/iso/2024.01.01/archlinux-bootstrap-x86_64.tar.gz
$ sha256sum arch*.tar.gz
```

## 持ってきたbootstrapのミラーリストを弄る
```bash
$ sudo tar zxvf arch*.tar.gz
$ sudo vim root.x86_64/etc/pacman.d/mirrorlist
```
適当なミラーリストのコメントアウトを外す．
つづけて圧縮作業
```bash
$ cd root.x86_64/
$ sudo tar czvf ../root.tar.gz ./*
```

## インポートする
Cドライブ直下に"wsl"という名のフォルダを作っておく

```powershell
PS C:\Users\windows-username\> wsl --import Archlinux C:\wsl\arch C:\Users\windows-username\Downloads\tmp\root.tar.gz
```

## Arch Linux on WSL2に入っての作業
```powershell
PS C:\Users\windows-username\> wsl -d Archlinux
```

### pacman周り
```bash
# pacman-key --init
# pacman-key --populate archlinux
# pacman -Syyu base base-devel git sudo vim 
```

### ルートパスワードの設定
```bash
# passwd
```

### ユーザ追加及びユーザをsudoersに設定
 - ユーザ追加について（[参照](https://github.com/Rintaro-T/Chirashi_Ura/blob/main/ArchLinux_install_Documents/Document_for_BIOS_machine.md#%E3%83%A6%E3%83%BC%E3%82%B6%E3%81%AE%E8%BF%BD%E5%8A%A0)）
 - sudoersについて（[参照](https://github.com/Rintaro-T/Chirashi_Ura/blob/main/ArchLinux_install_Documents/Document_for_BIOS_machine.md#%E4%BD%9C%E6%88%90%E3%81%97%E3%81%9F%E3%83%A6%E3%83%BC%E3%82%B6%E3%81%AB%E3%81%A6sudo%E3%82%92%E4%BD%BF%E3%81%88%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E3%81%99%E3%82%8B)）

### ロケールの設定（[参照](https://github.com/Rintaro-T/Chirashi_Ura/blob/main/ArchLinux_install_Documents/Document_for_BIOS_machine.md#%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB%E3%81%AE%E8%A8%AD%E5%AE%9A)）

### wsl.conf
archをwslコマンドから起動した際の諸々について"/etc/wsl.conf"に設定する．私の場合は下記
```
#systemdを使う
[boot]
systemd=true

#wslログイン時のユーザを指定する
[user]
default=rintaro-t

#メモリ上限を割り当てる
[wsl2]
memory=8GB
```

### その他個人設定
#### rintaro-tユーザのプロンプトを変更
```bash
# su rintaro-t
$ vim ~/.bashrc
```
該当箇所を下記のように変更
```
PS1='\[\e[32m\][\u@\[\e[31m\]\h \[\e[34m\] \w\[\e[32m\]]\n \$\[\e[m\] '
```
#### wslに入ったときにArchlinux上のホームディレクトリに行くようにする
```bash
$ echo 'cd ~' >> ~/.bashrc
```

## その他

### wslのアップデート
今回適当に作った後にArchLinux on wsl2上でsystemctlコマンドを叩いたところエラーを吐いた．
wslをアップデートしたら直った．
```powershell
PS C:\Users\windows-username\> wsl --update
```

### 規定ディストリビューションをArchlinuxにする
```powershell
PS C:\Users\windows-username\> wsl --set-default Archlinux
```
