---
title : "PIC16F84A : マトリクスLEDを使う(74HC595使用)"
tags : [
  "electronics",
  "pic16f84a",
  "matrix-led",
]
date : "2018-03-21T15:05:29+09:00"
---

16F84Aのコツをつかんできたため, 思い切って[8x8マトリクスLED](http://www.akiba-led.jp/product/1068)の制御をしてみた.
<!--more-->

{{< figure src="/img/posts/pic16f84a/matrix-1.jpg" title="" >}}

といっても, 16F84AにはI/Oピンが13本しかないので, 16本のマトリクスLEDを制御するために[シフトレジスタ(74HC595)](http://akizukidenshi.com/catalog/g/gI-08605/)を使用する.
例のごとくブレットボードにこの量の配線を毎回するのは嫌なので, いつもの[45x45mm基板](http://akizukidenshi.com/catalog/g/gP-11735/)に組みた. UEW配線初挑戦 ! 

{{< figure src="/img/posts/pic16f84a/matrix-2.png" title="回路図" >}}

基板の写真は[こちら](https://lh3.googleusercontent.com/-PZ5c4euMezw/WrI11CaCcZI/AAAAAAAAJCw/Dba29eeOBdYw3Wnz4I8xtu5J_V4SQRw-wCE0YBhgL/s1024/DSCF0286-COLLAGE.jpg).   
あと, Timer0を使ってより高度(?)な制御にも挑戦. 1msごとに一列ずつ表示することで自由に表示できるようにしてみたり.

{{< figure src="/img/posts/pic16f84a/matrix-3.jpg" title="丸のつもり" >}}

シャッタースピードを上げて撮るとこの通り. 一列ずつ表示されている.

{{< figure src="/img/posts/pic16f84a/matrix-4.jpg" title="シャッタースピードを上げて撮影" >}}

アニメーション（？）だってできる.

{{< tweet 976336217829552128 >}}

ソースコードは[こちら](https://gist.github.com/ha2zakura/fda8e0cb926c7603b90fe39fcf1147e4). main1.asmがトップ画像の表示で, main2.asmがダイヤみたいなやつ, main3.asmがアニメーション. 参考までに.   
