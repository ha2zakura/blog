---
title: "HugoでTikZを使う"
tags : [
  "hugo",
  "tikz",
]
date: 2020-04-14T18:10:05+09:00
toc : true
math: true
tikz: true
---

Hugoは$\KaTeX{}$で数学表記できるように[設定できる](https://katex.org/docs/autorender.html).
それで遊んでいたときに,
TikZによる作図もできると便利だなと思って調べてみたら
[TikZJax](http://tikzjax.com/)なるものを見つけたので,
これのショートコードを作成した.
<!--more-->

{{< tikz title="TikZによるドーナツ(Torus)" scale="2">}}
\draw (-1,0) to[bend left] (1,0);
\draw (-1.2,.1) to[bend right] (1.2,.1);
\draw[rotate=0] (0,0) ellipse (100pt and 50pt);
{{< /tikz >}}

## TikZとは

>**TikZ and PGF are TeX packages for creating graphics programmatically.**
>
>[TikZ and PGF | TeXample.net](http://www.texample.net/tikz/)

画像を$\TeX{}$で書ける便利なツール.
ここでは文法などの説明は割愛するので,
詳しいことは以下のQiita記事がわかりやすい.

[PGF/TikZをオススメする記事 - Qiita](https://qiita.com/seekworser/items/0ef417ab788e0786d59a)

結構メジャーなので日本語文献も多い.

### TikZJax

[kisonecat/tikzjax: TikZJax is TikZ running under WebAssembly in the browser](https://github.com/kisonecat/tikzjax)

TikZをブラウザ上で描画してしまうという恐しいツール.
日本語文献は限りなく無いが,
`README.md`によると
[kisonecat/web2js](https://github.com/kisonecat/web2js)なるものを使って,
WebAssemblyにコンパイルしているらしい.
更に恐しい. 使ってみたい.

ただし`\usetikzlibrary`には対応していない
(元コードに追加すればいける?)
ので注意が必要.

## Hugoで使う

[README.md](https://github.com/kisonecat/tikzjax/blob/master/README.md)の通りにやればできると思ったが,
途中でWebAssemblyをfetchする時に[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)で引っ掛かる.
なので, あまり上品ではないが, スクリプトとWebAssemblyを手元に落とすことにする.

まず問題のスクリプトを引っ張ってくる.

```bash
wget -O static/script/tikzjax.js http://tikzjax.com/v1/tikzjax.js
```

このスクリプトの**21864~21866行目**を以下のように変更.

```js
let tex = await fetch(urlRoot + '/wasm/tikzjax.wasm');
code = await tex.arrayBuffer();
let response = await fetch_readablestream__WEBPACK_IMPORTED_MODULE_5___default()(urlRoot + '/gzip/tikzjax.gz');
```

そしてWebAssembly達を引っ張ってくる.
セキュリティが心配な場合は`git clone`して手元でビルドする.

```bash
wget -O static/wasm/tikzjax.wasm http://tikzjax.com/ef253ef29e2f057334f77ead7f06ed8f22607d38.wasm
wget -O static/gzip/tikzjax.gz http://tikzjax.com/7620f557a41f2bf40820e76ba1fd4d89a484859d.gz
```

そして`layouts/partials/tikzjax.html`を作成.

```html
<link rel="stylesheet" type="text/css" href="http://tikzjax.com/v1/fonts.css">
<script src="/script/tikzjax.js"></script>
```

1行目は数式用のフォントを定義しているが,
BaKoMaフォントというものらしい.
[CTANにある](https://ctan.org/tex-archive/fonts/cm/ps-type1/bakoma/)ので
`static/font/`などに配置してもいいかもしれない.

あとは, レイアウトのhead部分に

```html
{{ if or .Params.tikz .Site.Params.tikz }}
  {{ partial "tikzjax.html" . }}
{{ end }}
```

などと追加する. 
これだと記事ファイルの冒頭部分に`tikz: true`とすることで
スクリプトが読み込まれる. ここら辺は好みであろう.

###  ショートコード

`layouts/shortcodes/tikz.html`を作成する.
これも好みだと思うが, 私は`{{ figure }}`風に登録している.

```html
<figure>
  <script type="text/tikz">
    \begin{tikzpicture}[{{ with .Get "scale" }}scale={{ . }}, transform shape{{ end }},{{ with .Get "domein" }}domain={{ . }}{{ end }}]
      {{ .Inner }}
    \end{tikzpicture}
  </script>
  {{ with .Get "title" }}
  <figcaption>
    <h4>{{ . }}</h4>
  </figcaption>
  {{ end }}
</figure>
```

これ以外に`<style>`~`</style>`でテーマに合うようにデザインを調整している.

## サンプル

[TEXample](http://www.texample.net/)にサンプルがあるのでいくつか書かせてみる.

### A simple cycle

[A simple cycle | TikZ example](http://www.texample.net/tikz/examples/cycle/)

{{< tikz title="A simple cycle" scale="1.5" >}}
\def \n {5}
\def \radius {3cm}
\def \margin {8} % margin in angles, depends on the radius

\foreach \s in {1,...,\n}
{
  \node[draw, circle] at ({360/\n * (\s - 1)}:\radius) {$\s$};
  \draw[->, >=latex] ({360/\n * (\s - 1)+\margin}:\radius) 
    arc ({360/\n * (\s - 1)+\margin}:{360/\n * (\s)-\margin}:\radius);
}
{{< /tikz >}}

```tex
{{</* tikz title="A simple cycle" scale="1.5" */>}}
\def \n {5}
\def \radius {3cm}
\def \margin {8} % margin in angles, depends on the radius

\foreach \s in {1,...,\n}
{
  \node[draw, circle] at ({360/\n * (\s - 1)}:\radius) {$\s$};
  \draw[->, >=latex] ({360/\n * (\s - 1)+\margin}:\radius) 
    arc ({360/\n * (\s - 1)+\margin}:{360/\n * (\s)-\margin}:\radius);
}
{{</* /tikz */>}}
```

### Intersecting lines

[Intersecting lines | TikZ example](http://www.texample.net/tikz/examples/intersecting-lines/)

{{< tikz title="Intersecting lines" scale="2.5" >}}
% Draw axes
\draw [<->,thick] (0,2) node (yaxis) [above] {$y$}
    |- (3,0) node (xaxis) [right] {$x$};
% Draw two intersecting lines
\draw (0,0) coordinate (a_1) -- (2,1.8) coordinate (a_2);
\draw (0,1.5) coordinate (b_1) -- (2.5,0) coordinate (b_2);
% Calculate the intersection of the lines a_1 -- a_2 and b_1 -- b_2
% and store the coordinate in c.
\coordinate (c) at (intersection of a_1--a_2 and b_1--b_2);
% Draw lines indicating intersection with y and x axis. Here we use
% the perpendicular coordinate system
\draw[dashed] (yaxis |- c) node[left] {$y'$}
    -| (xaxis -| c) node[below] {$x'$};
% Draw a dot to indicate intersection point
\fill[red] (c) circle (2pt);
{{< /tikz >}}

テーマと合わせるために`filter: invert()`で色を反転させているので,
丸が水色になっている.

```tex
{{</* tikz title="Intersecting lines" scale="2.5" */>}}
% Draw axes
\draw [<->,thick] (0,2) node (yaxis) [above] {$y$}
    |- (3,0) node (xaxis) [right] {$x$};
% Draw two intersecting lines
\draw (0,0) coordinate (a_1) -- (2,1.8) coordinate (a_2);
\draw (0,1.5) coordinate (b_1) -- (2.5,0) coordinate (b_2);
% Calculate the intersection of the lines a_1 -- a_2 and b_1 -- b_2
% and store the coordinate in c.
\coordinate (c) at (intersection of a_1--a_2 and b_1--b_2);
% Draw lines indicating intersection with y and x axis. Here we use
% the perpendicular coordinate system
\draw[dashed] (yaxis |- c) node[left] {$y'$}
    -| (xaxis -| c) node[below] {$x'$};
% Draw a dot to indicate intersection point
\fill[red] (c) circle (2pt);
{{</* /tikz */>}}
```
