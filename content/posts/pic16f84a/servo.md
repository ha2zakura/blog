+++
title = "PIC16F84A : サーボモータを使う"
tags = [
  "electronics",
  "pic16f84a",
]
date = "2018-03-20T16:26:07+09:00"
images = [
  "//drive.google.com/uc?export=view&id=1AGNdfBCHoitsCB0iixzLPH57Yclv-nVp",
]
+++

{{< figure src="//drive.google.com/uc?export=view&id=1AGNdfBCHoitsCB0iixzLPH57Yclv-nVp" title="" >}}

PIC16F84AでTimer0を使って，サーボモータを制御してみた．
Timer0で20msの周期で割り込みして，0.5~2.5msだけ出力をHIGHにする．
<!--more-->
一応，今回も[ソースコード](https://gist.github.com/ha2zakura/1ea44fd9f3cc07e0533092fa847bb244)をGistに公開するが，どこか絶対間違っている．特に20msの周期の計算は適当．まぁ動いたからOK...?

あと，サーボモータをただ動かすだけでは寂しいので，デジタル入力にも挑戦．デジタル入力は予想以上に簡単ね．もっと早くにやっておけばよかった... 

↓制御した証拠

{{< tweet 975984812916813825 >}}

デジタル入力のために，[16F84Aボード]({{< ref "/content/posts/pic16f84a/board.md" >}})を少し修正．2×3のピンヘッダはサーボモータ用だったが，そもそも4つもサーボモータを持ってないので，片方を取り除いてタクトスイッチを配置，ピンヘッダとジャンパピンも追加して，リセットボタンと併用にしてみたりと，いろいろ工夫してみた．  

{{< figure src="//drive.google.com/uc?export=view&id=1Qm67jrnlxf0Aq8SCR4Bj4syobZaXZkqy" title="後から修正すると回路図を書くのが面倒" >}}

