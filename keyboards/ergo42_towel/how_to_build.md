# Elite-Piを用いた Ergo42 Towelの作成ログ
 - 組み立てについては割愛
 - Ubuntu on WSL2を用いたファームウェアのビルドおよび焼きこみを記す

## 必要環境を準備する
qmkのインストールまで
```sh
$ sudo apt update && sudo apt upgrade
$ sudo apt install git python3 pip avrdude
$ python3 -m pip install --user qmk
```
PATHを通す
```sh
$ echo 'PATH="$PATH:~/.local.bin"'
$ bash
```

qmkのセットアップ
この際大体y押しておけばおっけ
```sh
$ qmk setup
```

## Elite-Pi向けのファームウェアの作成/
keymap.cを~/qmk\_firmware/keyboards/biacco42/ergo42/keymaps/default下にコピー
ただし，keymap.cの書き方が変わっていたりするので見比べて書き方の確認はしたほうがよさげ

ビルド
```sh
$ qmk compile -c -kb biacco42/ergo42 -km default -e CONVERT_TO=elite_pi
```
~/qmk\_firmware/.build下にuf2ファイルが作成されている

## 焼きこみ
先ほどのファイルをとりまデスクトップにコピー
```sh
$ cp ~/qmk_firmware/.build/*.uf2 /mnt/c/User/$USERNAME/Desktop
```
あとはElite-PiをPCにつないで
 - 初回なら勝手にフォルダが表示される
 - ファームの書き換えならErgo42のリセットボタンをダブルクリックの要領で2度押す
で表示されたフォルダに先のuf2ファイルをD&Dで終了
Ergo42 Towelは右左どちらのElite-Piも天地反転で取り付ける
