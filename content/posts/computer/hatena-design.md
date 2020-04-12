+++
title = "はてなブログのデザインを一新する"
tags = [
  "computer",
  "hatena",
]
date = "2020-03-04T17:41:58+09:00"
images = [
  "//drive.google.com/uc?export=view&id=1WoS5J11xSG46pN655e5FTiN5ifDdzcm7",
]
+++

久々にはてなブログのデザインを更新した．
<!--more-->
前のデザインを作ったのは3年前の[はてなブログに引っ越してきた時](https://ha2.hateblo.jp/entry/start)で，
それ以降はメニューバーのマイナーチェンジを繰り返したり，
レスポンシブデザインをやめたりした程度で，
はてなブログに来てからデザインを一新したのは今回が始めて．

新デザインを思った以上に気に入っているので，
これができるまでの過程とか苦労をまとめておきます．

# 構想

前回のデザインもそこそこ気に入っていた．
あのシンプルさと見易さははてなブログにしては
いいデザインにできたと思っている(失礼)．
しかし，そこはかとなく感じられる**古さ**と
誤魔化しきれなくなってきた**不具合**が重なり，
今回のデザイン一新を決意した．

新しいデザインは前回のシンプルさを踏襲しつつ，
しかし現代的なデザインで統一しようと思った．
また，3年使ってみて必要に感じた，
機能面での改良もしたいと考えた．
そこで，今回始めて**モックアップ**を作ってみた．

## モックアップ

{{< figure src="//drive.google.com/uc?export=view&id=1QvEJgNgQ3yezVJws4DaQdBOh9ViKL7kA" title="スクリーンショット" >}}

[Moqups](https://moqups.com/)という
モックアップ・ワイヤーフレーム製作ツールを使って
完成イメージを(細かく)作った．
このツールの良いところは，
CSSを意識したパラメータなのに直感的に扱い易く
初見でもすぐに慣れることができた．
(はてなのバーはスクショしたものを貼り付けた．
再現は大事)

このツールの欠点は，**無料だと出力できない**こと．
しかし，裏紙とペンで古代の壁画を描きながら製作していた
従来よりは格段に楽．
これで全体から統括的にデザインすることができた．

新デザインでは，サイドメニューを廃止，
その空いたところにTwitterへのリンクと
目次を配置する．
前者は海外ニュースサイトのタイトル下記者紹介，
後者は[Qiita](https://qiita.com)など技術系ナレッジ共有サイトをイメージしてます．

# 製作

今回はデザインを一新するということで，
コードも0から全て書き直した．

## Boilerplateは使わない

[hatena/Hatena-Blog-Theme-Boilerplate](https://github.com/hatena/Hatena-Blog-Theme-Boilerplate)

はてなブログ運営は良心的で，
テーマ製作のための土台となるサンプルテーマ**Boilerplate**が配布されている(しっかりMITライセンス)．
これはデザインの最低限が書かれており，
前回のテーマもこれを下敷に製作した．

しかし，今回は0から秩序的に構築するために，
Boilerplateは環境のみ使わせて頂きた．

## 設計(粒度分類)

[脱・Atomic Design - HTML+CSSコーディングの粒度分類法（HTML Parts）](https://qiita.com/croco_works/items/e34d1b0c0e50b37031d7)

たまたまこの記事を読んだばかりであったので，
今回は**粒度分類**をある程度踏まえてコード設計をした．

### Block

まず，上のモックアップよりBlockを分けます．

- サイト全体
  - ヘッダ
  - ヘッダ下メニュー(独自HTML)
  - 記事
  - フッタ上メニュー(独自HTML)
  - フッタ

前述のTwitterへのリンクと目次は記事Block内に含みます．
つまり，大きく見れば1カラムになる．
このBlockごとにSassファイルを分け，
`scss/parts/` フォルダにまとめた．

### Module・Component

次にModule．
と言っても，Moduleになるのは記事Block程度なので，
Compornentの話も含みます．
記事以外のBlockはModule・Componentを意識せず
Elementに切り分けてすぐに実装できた．
独自HTMLについては[CSSのセレクタが右から読まれる](https://qiita.com/Yametaro/items/5c2d0267156ecc8562e4)ことを意識して，
BEMっぽい命名にする(初挑戦)．

記事Blockについては，

- 記事Module(一つの記事)
  - 記事ヘッダ
  - 記事筆者(Twitterへのリンク)
  - 記事目次
  - 記事本体
  - 記事フッタ

と分けた．
ここで問題となるのが，
元々のHTMLには無い**記事筆者**と**記事目次**の追加．
特に後者は元々記事本体Component内にあるものを外に出す必要があるので，
`window.onload`におけるDOMで2つのComponentを追加する．

## HTML・JS実装

独自HTMLの「ヘッダ下メニュー」と「フッタ下メニュー」は
普通に実装した．

### 記事筆者

まず，以下の記事筆者のHTMLを，

```html
<div class="entry-author">
  <a href="https://twitter.com/TsL_26">
    <img src="https://pbs.twimg.com/profile_images/1227453503032975362/iOceacnm_200x200.jpg" alt="ha2zakura" class="entry-author-img">
  </a>
  <div class="entry-author-name">
    <p class="entry-author-name-p">ha2zakura</p>
    <a href="https://twitter.com/TsL_26" class="entry-author-name-a">TsL_26</a>
  </div>
</div>
```

これを変数(`author_html`)に格納する．

次に，記事(`.entry`)のHTMLCollentionを取得し，
記事ごとに目次を作成するために周回させます．

```javascript
const entry = document.getElementsByClassName("entry"); // 記事のHTMLCollection
if (entry)  // Archiveページ避け
  for (var i = 0; i < entry.length; i++) {

    // 記事ごとの処理

  }
```

一応，Archiveページなどで記事が無いことが考えられるので，
`entry`が空でないことを確認してから周回させてます．

後は記事ヘッダを取得して，記事ヘッダの直後に`author_html`を追加すれば完了．

```javascript
// 記事ヘッダの取得
const entry_header = entry[i].getElementsByClassName("entry-header");
// 記事ヘッダの直後にauthor_htmlを追加(1つのentry内に記事ヘッダは1つ)
entry_header[0].insertAdjacentHTML("afterend", author_html);
```

### 記事目次

記事筆者Componentと同じように，記事ごとに処理させます．

まず，記事本体Component内にある元々の目次を取得する．

```javascript
// 記事目次(.table-of-contents)
const table_of_contents = entry[i].getElementsByClassName("table-of-contents");
```

ここで，記事目次内の`a`タグに独自のClass(`.entry-index-a`)を付け加える．(理由は後述)

```javascript
// 記事目次のaタグに.entry-index-aを追加
for(var j = 0; j < table_of_contents[0].getElementsByTagName("a").length; j++)
  table_of_contents[0].getElementsByTagName("a")[j].classList.add("entry-index-a");
```

そして，これを`div.entry-index`に格納して記事ヘッダ直後に追加させれば目次を配置できる．

```javascript
// 記事目次を格納するdivを用意
const entry_index = document.createElement('div');
// Class名を付加
entry_index.className = "entry-index";
// 取得した記事目次を格納
entry_index.appendChild(table_of_contents[0]);
// 記事ヘッダの直後に追加
entry_header[0].insertAdjacentElement("afterend", entry_index);
```

#### 現在位置を目次に表示する

せっかく記事目次を外に出したので，
Qiitaのように「自分が今どのあたりを読んでいるのか」を表示する．

まず，記事内の見出し(`h1`とか)を探し出して独自Class(`.entry-content-h`)を付加する．

```javascript
// 記事本文(.entry-content)
const entry_content = entry[i].getElementsByClassName("entry-content")[0];
// 見出しを探し出して.entry-content-hを付加
for (var j = 0; j < entry_content.children.length; j++) {
  var entry_content_child = entry_content.children[j];
  entry_content_child.classList.add(
    ["h1", "h2", "h3", "h4", "h5", "h6"].includes(entry_content_child.toLowerCase()) ?
      "entry-content-h" : "entry-content-not-h"
  );
}
```

こうすることによって，
`document.getElementsByClassName("entry-content-h")`で
ページ内にある全てのページ見出しのHTMLCollectionを取得できる．

これと前述の`.entry-index-a`を使って，
見ている範囲の直上にいる`.entry-content-h`の要素の添字と同じ添字の
`.entry-index-a`の要素に対して，
それを知らせるClass(`.focus`)を付加すれば目的は達成できる．

```javascript
// 記事目次のFocus
const index_focus = () => {
  // .entry-content-h取得
  const entry_content_h = document.getElementsByClassName("entry-content-h");
  // .entry-index-a取得
  const entry_index_a = document.getElementsByClassName("entry-index-a");

  var place_flag = false;   // 現在位置検知フラグ
  if (entry_content_h)  // 見出しが無い記事避け
    for (var i = entry_content_h.length - 1; i >= 0 ; i--) {  //後ろから探索
      // 画面上端より上にある1個目の見出しのみ.focusを付加
      if (entry_content_h[i].getBoundingClientRect().top > 50 || place_flag)
        entry_index_a[i].classList.remove("focus");
      else {
        entry_index_a[i].classList.add("focus");
        place_flag = true;
      }
    }
}
```

この関数`index_focus()`をscrollイベントの度に発火させれば
現在位置が常に表示され続ける．
しかし，毎回探索させると画面がガタつくので，
`setTimeout(index_focus()，1000/60)`とかを上手く使って
発火を間引く．
詳しくは[ここ](https://qiita.com/kikuchi_hiroyuki/items/7ac41f58891d96951fa1)を参考にした．

なお，**「見出しはあるけど目次のない記事」ではエラーが出て上手く表示されない**．
避けを作っても良かったのだが，
このデザインだとむしろ目次は必須であるため
訂正していない．

この目次の現在位置表示，
以外と情報が少なかったので0から実装した．
ここの情報が少しでも参考になれば...

## CSS装飾

もっとSassらしい構成にするべきだったのでしょうが，
前述のファイル分割以外はいつもの癖で`.css`っぽい書き方になった．

### タイトル

そういえばブログタイトルを変えた．
旧タイトル**のんびり努力中**は小学校時代のブログ開設時から
使っていた名前なので名残り惜しいが，
ぶっちゃけデザインに合わなかったので変えた．


で，上のタイトルにあるように，
ラテン字だと長いので2色に分けたのが，
はてなではタイトル部分のHTMLをいじることができません．
なので，`:before`と`:after`を使って上から文字を重ねます．

```css
#title a {
  color: #ffffff00;
  position: relative;
}

#title a:before {
  content: 'NOMBIRI';
  overflow: hidden;
  position: absolute;
  left: 0;
  top: 0;
  color: #444;
}

#title a:after {
  content: 'doryoku';
  overflow: hidden;
  position: absolute;
  right: 0;
  top: 0;
  color: #888;
}
```

こうすることで，見かけ上は`NOMBIRI`と`doryoku`の
2色に分かれる．
これの欠点は，タイトルを変えたときは
`content` も変えなければならないところ．
まあ，もうしばらくは変えないでしょう，きっと．

### レイアウト

前述の通り，全体としては1カラムなので単純に並べられます．
しかし，記事Moduleは
記事筆者・目次と，記事ヘッダ・本体・フッタの
2カラム風デザインにする必要があった．

前回の2カラムレイアウトは悪名高い `float: left` とか `clear: both` とかを
使って実装した．
が，前回の製作から3年，いつのまに
[Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout)なるものが
普及してた．
今回はこれを使って実装する．

```css
.entry-inner {
  display: grid;
  grid-template-columns: 1fr 300px;
  /* 2カラムの配置を指定 */
  grid-template-areas: 
    "header author"
    "content index"
    "footer .";
}

/* 各要素にgrid-areaで位置を示してあげる */
.entry-header {
  grid-area: header;
  /* ... */
}

.entry-content {
  grid-area: content;
  /* ... */
}

/* ... */
```

これだけで記事ごとに2カラムが出来上がります．
それぞれのカラムの幅も `grid-template-columns` で設定するだけ．
世の中は便利になっていくもんね...

#### 記事目次を画面にくっつける

なんのことはない，

```css
.table-of-contents {
  position: sticky;
  /* ... */
}
```

これで一発．

### 読み込み画面

JSを増やしすぎたせいで，
読み込みがとても遅くなった．
しかも，はてなブログは写真のサイズを落とさないために
結構気になるレベルで遅い．

なので，誤魔化すために読み込み画面を追加した．

```html
<div id="loader-outer" class="show">
  <!-- 読み込み中のアニメ -->
</div>
```

読み込みのアニメは「css loader animation」で検索すると出てくる．
はてなブログでは[こちら](https://tobiasahlin.com/spinkit/)を使わせて頂いている．

全画面にする方法は今更書く必要も無さそうだが，
意外と忘れやすいので置いておく．

```css
#loader-outer {
  opacity: 0;
  z-index: 100;
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  background: #fff;
  overflow: auto;
  visibility: hidden;
}

#loader-outer.show {
  opacity: 1;
  visibility: visible;
}
```

で，これを解除する関数，

```javascript
// ローダーの解除
const loader_remove = () =>
  document.getElementById("loader-outer").classList.remove("show");
```

関数`loader_remove()`を`window.onload`の最終行に追加してあげれば完成．

読み込み画面を作るだけで体感的には読み込みが早くなった．
しかし，なんの解決にもなっていない...

# ソースコード

[ha2zakura/Hatena-Blog-Theme-Shimakaze](https://github.com/ha2zakura/Hatena-Blog-Theme-Shimakaze)

今回のデザインのソースコード．  
本当ははてなブログのテーマとして公開しようとしていたが，
JSで独自Classを付加しすぎたのでやめにした．
なので，あくまで**参考**にしてください．

なお，引用とか私のブログで使わない要素に関しては
まったくノータッチ．
