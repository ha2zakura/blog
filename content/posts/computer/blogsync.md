---
title : "記事管理にblogsyncを導入した"
tags : [
  "computer",
  "hatena",
]
date : "2020-01-02T15:06:53+09:00"
images : [
  "//drive.google.com/uc?export=view&id=1WoS5J11xSG46pN655e5FTiN5ifDdzcm7",
]
toc : true
---

はてなブログを更新する時の1番の障壁って，ずばり，ブラウザエディタよね．書き辛いのなんの．

で，色々調べてたら `blogsync` なるものを見付けたので，導入してみた．
<!--more-->

## blogsyncとは

これ．[x-motemen/blogsync: Push and pull blog entries from/to local filesystem](https://github.com/motemen/blogsync)

Go言語で書かれた，「はてなBlog用のCLIクライアント」．こんな便利なものがあるとは．．．

これを用いることで，ローカル上で記事の作成，投稿ができる．  
なにがいいって，VSCodeで記事が編集できる!

## 導入

導入方法は[README](https://github.com/motemen/blogsync/blob/master/README.md)に書かれている．これを基に，少し気になったことなど．

### インストール

#### macOS

```bash
$ brew install Songmu/tap/blogsync
```

[Homebrew](https://brew.sh/)を使う場合．学校でも `brew` を使いたい．
愚痴はさておき，私の環境はmacOSではないので，次へ．

#### Ubuntu / WSL

``` bash
# go が使えない場合
$ sudo snap install --classic go
$ echo "export GOPATH=$HOME/.go" >> ~/.bashrc
$ echo "export PATH=$PATH:$GOPATH/bin" >> ~./bashrc

# blogsync のインストール
$ go get github.com/motemen/blogsync
```
前半の`go`のインストールで悩んだ．`$GOPATH`の設定がなければいけないよう．`go`のインストールについては[Go言語のインストール - golang.jp](http://golang.jp/install#freebsd_linux)なども参照してください．

### 設定ファイル

`blogsync` を使うための設定ファイルを `~/.config/blogsync/config.yaml` に作成する．

```yaml
<キー>:
  username: myname
  password: <API KEY>
  local_root: /home/mmyname/Blog
  omit_domain: true
```

設定する項目は，

- キー : ブログのドメイン(`ha2.hateblo.jp` など)．ただし，独自ドメインを設定している場合は独自ドメインにする前のドメインにする必要がある．
  - `username`: はてなユーザのID．
  - `password`: 投稿するためのAtomPubのAPIキー．ブログの[詳細設定画面](http://blog.hatena.ne.jp/my/config/detail)から確認できる．
  - `local_root`: エントリを格納するパスのル^ト．
  - `omit_domain`: パスにドメインを含めないか．`false` にすると `(local_root)/mine.hateblo.com/entry` のようにキーのフォルダが挟まる．
- `default`: すべてのブログの項目のデフォルト値．

最後の `default` は複数のブログを管理するときに便利．例えば，

```yaml
mine1.hateblo.jp:
  username: myname1
  password: <API KEY1>
mine2.hateblo.jp:
  username: myname2
  password: <API KEY2>
default:
  local_root: /home/mmyname/Blog
  omit_domain: false
```

とすることで，

```
/home/mmyname/Blog/
├── mine1.hateblo.jp
│   └── entry
└── mine2.hateblo.jp
    └── entry
```

となる．

## 使う

### エントリのダウンロード

```
$ blogsync pull <キー>
```

で，エントリを `local_root` にダウンロードできる．フォルダ構成は記事URLの設定によって異なる．

### エントリの形式

エントリは[YAML Frontmatter](https://assemble.io/docs/YAML-front-matter.html)になっていて，拡張子は `.md`．たとえば，この記事の下書き時点での冒頭部分は，

```yaml
---
Title: blogsyncを導入した
Category:
- Computer
Date: 2020-01-02T15:06:53+09:00
URL: https://ha2.hateblo.jp/entry/2020/01/02/150653
EditURL: https://blog.hatena.ne.jp/ha2zakura/ha2.hateblo.jp/atom/entry/26006613492177787
Draft: true
toc : true
---

ブログを更新する時の1番の障壁って，…
```

となる．各項目の内容は雰囲気でわかる．

### エントリの更新

```bash
$ blogsync push <エントリのパス>
```

`blogsync push` で下書きや既に投稿した記事を更新できる．

### エントリの投稿

```bash
$ blogsync post <キー> [--draft] [--title=<タイトル>] < <エントリの内容(.txtとか)>
```

`blogsync post` は標準入力からエントリの内容を受け取って投稿し，そのエントリをダウンロードする．

引数は，

- `--draft`: 下書き記事として投稿する．
- `--title=<タイトル>`: 記事のタイトルを指定する．

### エントリの作成

`blogsync` にはエントリ作成にあたるコマンドは無いが，`blogsync post`を応用して記事を作成できる．

```bash
$ echo '' | blogsync post <キー> [--draft] [--title=<タイトル>]
```

これによって `local_root` に新しいエントリが追加されるので，これを編集して `blogsync push <パス>` する．

## 思ったこと

VSCodeで記事を編集し，GitHubでバージョン管理．  
これで書く抵抗もなくなって...

と思いきや，実は以外と気になることがある．

- `blogsync push` で下書きを更新するとエントリがダウンロードされ直す．
- 記事URLの設定の設定によってエントリのフォルダ構成が違い，途中で変えたりしてるとごちゃごちゃに．
- マークダウン記法でも，はてなブログの独自記述(リンクとか)がVSCodeのプレビューでわからない．

あと，

[コード化したはてなブログリポジトリの更新を CircleCI 2.0 で自動化した - えいのうにっき](https://blog.a-know.me/entry/2018/03/04/215345)

`blogsync`の挙動すらCircleCIで自動化している人がいた．すごい．しかし，

[令和元年冬 犬は駆け廻り 猫は円くなり 私はCIに目を回す - Qiita](https://qiita.com/sugimoto_teco/items/e5431c6bb40b662f2f93)

どうやら2019年12月時点でどうにも使えないらしい...
