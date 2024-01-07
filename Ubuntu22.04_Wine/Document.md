# Ubuntu22.04にてWineを用いて某ゲームをやるための設定
条件としてはVMware Workstation Player17を用いている

## 適当にUbuntuをインストール
※省略

## Wineをインストール
```bash
$ sudo apt update && sudo apt upgrade
$ sudo apt install wine winetricks
```

## OPムービーを再生できるようにする
```bash
$ winetricks quartz wmp9 wmv9vcm
```

## ゲームのインストール
あとはWine使ってインストールしてやる
細かいところは省略
