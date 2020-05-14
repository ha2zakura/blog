---
title: "Raspberry Pi Zero W + Git + Dockerで環境構築"
date: 2020-05-14T13:01:47+09:00
toc : true
tags : [
  'computer'
]
---

約1年前に[Raspberry Pi Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/)を買ったものの,
環境を汚しまくって使わなくなった.
最近, 環境をDockerで整えることにハマっているので,
GitとDockerを使ってHerokuライクに環境構築する.
私は潔癖症である.

### 目次

1. [構成](http://localhost:1313/posts/computer/pi-docker/#%E6%A7%8B%E6%88%90)
1. [Raspbianの構築](http://localhost:1313/posts/computer/pi-docker/#raspbian%E3%81%AE%E6%A7%8B%E7%AF%89)
1. [DockerとDocer Composeの構築](http://localhost:1313/posts/computer/pi-docker/#docker%E3%81%A8docer-compose%E3%81%AE%E6%A7%8B%E7%AF%89)
1. [Gitサーバーの構築](http://localhost:1313/posts/computer/pi-docker/#git%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%81%AE%E6%A7%8B%E7%AF%89)
1. [Hello, world](http://localhost:1313/posts/computer/pi-docker/#hello-world)
1. [参考](http://localhost:1313/posts/computer/pi-docker/#%E5%8F%82%E8%80%83)

半分備忘録なので,
素の状態から構築する手順を載せる.

## 構成

開発の手順は[Heroku](https:/heroku.com)をイメージしている.
というかHerokuでもたぶん同じような技術が用いられている.

```
+-Raspberry Pi Zero W-------------------+
| *--Remote--+    +----Production-----+ |
| |          |pull|     +-Container-+ | |
| |   (2)    |===>| (3) |    (4)    | | |
| |          |    |     +-----------+ | |
| +----/\----+    +-------------------+ |
+------||-------------------------------+
       ||Push
  +--Local---+
  |   (1)    |
  +----------+

(1) ローカルリポジトリ, ここで開発
(2) リモートリポジトリ, ここにpush
(3) 本番環境, (2)へのpush後に自動的にpull
(4) デプロイ環境(コンテナ), (3)のpull後に走る
```

## Raspbianの構築

Debianで構築をする.
UbuntuやWindows, macOSでの構築は
別途検索されたい.

### Raspbianのインストール

Raspbianのイメージは
[公式サイト](https://www.raspberrypi.org/downloads/raspbian/)よりできる.
Desktop版でもいいと思うが,
Zero WではLite版で十分と思う.

SDカードへの書き込みは`dd`を使う.

```bash
sudo dd bs=4M if=2020-02-13-raspbian-buster-lite.img of=/dev/sdb conv=fsync
```

完了したら, SDカードの **boot** パーティションをマウントし,
USB経由でSSH接続できるように以下の操作をする.

```bash
# in boot/
touch ssh # SSH有効化
echo "dtoverlay=dwc2" >> config.txt
vim cmdline.txt
# modules-load=dwc2,g_ether を rootwait の直後に追加
# See also http://blog.gbaman.info/?p=791
```

SDカードをアンマウントし, Raspberry Pi Zero Wに挿入する.
USBとPCと繋げば数分でRaspbianが起動する.

### USB経由でSSH接続

Raspbianが起動してから
PCのネット設定を見ると
**Wired connection 1** (有線接続1) が追加されているので,
それのIPv4のMethodを **Link-Local Only** に変更し保存する.
たしかUbuntuでも同じような操作でできた記憶がある.

変更ができたら`ssh`コマンドで接続できる.

```bash
# On PC
ssh pi@raspberrypi.local
# Pass: raspberry

#-> pi@raspberrypi.local:~ $
```

先に鍵の設定をする.
クライアント側で`ssh-keygen`で鍵を作成し,
`ssh-copy-id`で鍵をラズパイに送信する.

```bash
# On PC
# 鍵作成
ssh-keygen -t rsa -b 4096 -f ~/.ssh/raspi

# 鍵送信
ssh-copy-id -i ~/.ssh/raspi.pub pi@raspberrypi.local
```

`/etc/ssh/sshd_config`に`PasswordAuthentication no`を追加すれば
鍵以外での接続を拒否できるようになるが,
この後Git用のユーザーを追加するときに
`ssh-copy-id`が使えなくなり面倒なので後にまわす.

#### .ssh/configの設定

`.ssh/config`に以下を追加すれば,
`ssh raspi`で接続できるようになる.

```
Host raspi
    HostName (hostname).local
    IdentityFile ~/.ssh/raspi
    User pi
    IdentitiesOnly yes
```

ホスト名 (`(hostname)`) は後で再設定するが,
変えない場合は`raspberrypi`になる.

### Raspberry Piの初期設定

SSHで接続.
ホスト名と, Wi-Fiの設定をする.

```bash
# Password, Host, Wi-Fi などの設定
sudo raspi-config
```

再起動する前に,
パッケージとOS, ファームウェアのアップデートをしておく.

```bash
# Packages と OS のアップグレード
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y

# Farmware のアップグレード
sudo rpi-update
sudo reboot # 再起動
```

## DockerとDocer Composeの構築

### Docker

Dockerをインストールし,
`sudo`無しで実行できるようにグループにユーザーを追加する.

```bash
# Docker のインストール
curl -sSL https://get.docker.com/ | sh

# "docker" グループにユーザーを追加
sudo usermod -aG docker pi
```

早速`docker run hello-world`でDockerを満喫したいが,
何も表示されなかった.
ARM6に対応していない可能性あり?
まあ`hello-world`が動かなくても本題には関係ない.

### Docker Compose

Docker ComposeはARM版のバイナリが公式サイトには転がっていない.
なので, `pip`よりインストールをする.
`python3-distutils`は`pip`をインストールする際に必要.

```bash
# pip のインストール
sudo apt-get install -y python3-distutils
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && sudo python3 get-pip.py

# Docker Compose のインストール
sudo pip install docker-compose
```

## Gitサーバーの構築

いまいち呼び方があってるかわからないが,
Gitサーバーを構築する.
Gitサーバーが何なのかは割愛.

まず, `git`のインストール.

```bash
# Git のインストール
sudo apt install -y git
```

### ユーザーの追加

Gitサーバーを扱う`git`ユーザーを追加する.
Dockerを使うために`docker`グループに追加もしておく.

```bash
# git ユーザーの追加
sudo adduser git
# Password を設定する

# "docker" グループにユーザーを追加
sudo usermod -aG docker git
```

ついでに, SSH接続のための鍵を登録する.
`pi`と同じ鍵で良いだろう.

```bash
# On PC
# 鍵送信
ssh-copy-id -i ~/.ssh/raspi.pub git@(hostname).local
```

これもまた`.ssh/config`に登録しておく.

```
Host gitpi
    HostName (hostname).local
    User pi
    IdentityFile ~/.ssh/raspi
    IdentitiesOnly yes
```

あと, `git`のための公開鍵も登録しておく.
GitHubなんかと同じ仕組みである.

```bash
# On PC
# 鍵が無かったら作成(~/.ssh/id_rsa & ~/.ssh/id_rsa.pub)
ssh-keygen -t rsa -b 4096 -C "email@example.com"

# 鍵送信
scp ~/.ssh/id_rsa.pub gitpi:.ssh
```

```bash
# On git@(hostname).local
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

長かったが,
これで環境は整った.

## Hello, world!

せっかくなので`nginx`を使ったHello, world!をする.

ソースのルートパスに`docker-compose.yml`を
ビルド, 実行するようにする.

### 新しいリポジトリの作成

Raspberry側に`--bare`なリポジトリと
本番用のリポジトリを作成する.

割と手数が多く,
スクリプトを組んだ方が良いかもしれない.

```bash
# On git@(hostname).local
# リポジトリの作成
cd ~
mkdir hello.git
cd hello.git
git init --bare

# Push時hookの追加
vi hooks/post-receive
```

```sh
#!/bin/sh

cd ~/prod/hello
git --git-dir=.git pull
docker-compose build
docker-compose pull
docker-compose up -d
```

```bash
# On git@(hostname).local
chmod +x hooks/post-receive

# 本番環境の作成
mkdir ~/prod
cd ~/prod
git clone ~/hello.git
```

### ローカルリポジトリからPushでデプロイ

GitHub - [ha2zakura/pidocker-server](https://github.com/ha2zakura/pidocker-server)

```bash
# On PC
git clone https://github.com/ha2zakura/pidocker-server.git
cd pidocker-server
git remote add pi git@(host).local:hello.git
git push pi master
```

`http://(hostname).local:8080/`にアクセスすれば,
**Hello, world!** が表示される.

## 参考

### Raspbian初期設定とか

- [Raspberry Pi ZeroをUSBケーブル1本で遊ぶ | Japanese Raspberry Pi Users Group](https://www.raspi.jp/2016/07/pizero-usb-otg/)

- [Raspberry Pi Zero – Programming over USB! (Part 2) | Andrew's blog](http://blog.gbaman.info/?p=791)

### Docker on Raspberry Pi

- [Raspberry PiにDockerを入れる - Qiita](https://qiita.com/hisurga/items/7aca7484ac5bfd084294)

- [Docker on Raspberry PiのインストールとLチカ - Qiita](https://qiita.com/ykshr/items/c78eb72e3ee75664a5fe)

- [Docker comes to Raspberry Pi - Raspberry Pi](https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/)

### Docker Compose on Raspberry Pi

- [Install Docker and Docker-Compose on your Raspberry Pi - Jonathan Meier](https://jonathanmeier.io/install-docker-and-docker-compose-raspberry-pi/)

### Gitサーバー

- [gitでシンプルなデプロイ環境を作る - Qiita](https://qiita.com/zaburo/items/8886be1a733aaf581045)

- [gitを使ったデプロイ方法 - Qiita](https://qiita.com/__mick/items/1f73fcb0d5dc7f5982f4)

- [Gitで基本的なデプロイ（push、pullで本番公開）環境を作る手順解説 | エス技研](https://blog.s-giken.net/343.html)

- [git pushで本番環境に"自動デプロイ"できる環境を作ってみよう！ | vdeep](http://vdeep.net/git-push-deploy)

### Nginx構築

- [Debian 9 (Stretch) - Web サーバ Nginx 構築（Nginx 公式リポジトリ使用）！ - mk-mode BLOG](https://www.mk-mode.com/blog/2017/09/16/debian-9-nginx-installation-by-official-apt/)
