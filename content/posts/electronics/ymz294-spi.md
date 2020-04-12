---
title : "YMZ294をSPIで制御する"
tags : [
  "electronics",
  "ymz294",
]
date : "2018-12-31T09:08:01+09:00"
toc : true
---

{{< figure src="//drive.google.com/uc?export=view&id=1dncsx3lp7OhynGFnx0cRKJZtXjJ6nOZb" title="" >}}

前に使った[YMZ294]({{< ref "/content/posts/electronics/ymz294-spi.md" >}})が，
また使おうとするとあの配線をやり直す必要があった．
そこで，思い切って[SPI](https://ja.wikipedia.org/wiki/%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%AB%E3%83%BB%E3%83%9A%E3%83%AA%E3%83%95%E3%82%A7%E3%83%A9%E3%83%AB%E3%83%BB%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%95%E3%82%A7%E3%83%BC%E3%82%B9)化することで配線の手間を減らし使いやすくしようと思い，やってみた．  
<!--more-->

## 変換アダプタもどき

{{< figure src="//drive.google.com/uc?export=view&id=1MHIMI546LFAj0alpA-9Du0WltGZ9Xn8R" title="変換アダプタ" >}}

SPI化には，[シフトレジスタ(74HC595)](http://akizukidenshi.com/catalog/g/gI-08605/)を用いた．
理由は，[前に使ったことがあるから]({{< ref "/content/posts/pic16f84a/matrix.md" >}})かね．
そのまま8ビットバスとSPIマスタの間につなげば使える，と思ったのが，
どうも書き込みの際の/WR/CSや/A0の扱いが74HC595だけでは上手くいきそうになかったので，
[PIC12F683](http://akizukidenshi.com/catalog/g/gI-00801/)で制御した．
74HC595がラッチされるのを検知して，/WR/CSをHIGH，LOWする．
PIC12F683の[ソースコード](https://gist.github.com/ha2zakura/943d75f813021092b6f4207b43e41da1)．
A0はマスタ側で直接制御する．ホントはPIC12F683で一緒に制御できれば楽だったんけどね...  

{{< figure src="//drive.google.com/uc?export=view&id=1mGEYMFKunaKNbyg64KEvP7A7_eAi6SmN" title="回路図" >}}

[裏面の写真](https://drive.google.com/uc?export=view&id=1FQ-ZkQqab4ziOb6fYMW-gDd6vvcevCin)

## 演奏してみた

割とMIDIでYMZ294を制御している方は多い．
中には私と同じようにピン数圧縮してる方もいらっしゃいた
（[YMZ294のMIDI音源化と，YMZのピン数圧縮 | curious4dev](http://curious4dev.mydns.jp/post-1184/)）．
似たようなこと考える人ってやっぱいる．  

ま，ピアノできないんでネットで拾ってきたMIDI譜面を流し込むだけなんけどね．  

{{< figure src="//drive.google.com/uc?export=view&id=1fcSoUZYeYQ9_9hFnOS2FGHZs8yUjy_7w" title="Arduinoと接続" >}}


マスタはArduinoを使う．理由はゴニョゴニョ...
（いつかはPIC18F14K50でMIDIを制御したい今日この頃．）  
[ソースコード](https://gist.github.com/ha2zakura/003ec66c355f43848eb86c17d77cc453)．
ネットのサンプルコードを組み合わせて適当に作ったので参考にもならないと思う．
（てか書いたの数か月前だから全然覚えていない...）  

MIDIの再生は[Domino](http://takabosoft.com/domino)で，MIDIデータの送信は，
本当は[Moco](http://morecatlab.akiba.coocan.jp/lab/index.php/aruino/midi-firmware-for-arduino-uno-moco/)を使いたかったけれど，
どうも前に色々あってArduino UNOのパターンをはがしてしまったようでDFUで認識できず．
かわりに[The Hairless MIDI<->Serial Bridge](http://projectgus.github.io/hairless-midiserial/)を使う．
先人に感謝 ! こちらはシリアル通信でMIDIが使えるため，互換機でも使える．

で，以下が演奏してみたもの．譜面は[かきむき様のもの](http://kakimuki.blog91.fc2.com/blog-entry-6.html)を使わせていただいてます．

[千本桜.mp3](https://www.dropbox.com/s/ingke1lf3zb9ivt/%E5%8D%83%E6%9C%AC%E6%A1%9C.mp3?dl=0)

[Bad_Apple!!.mp3](https://www.dropbox.com/s/kxlt83dpht7spfb/Bad_Apple%21%21.mp3?dl=0)

[Dropbox - Tell_Your_world.mp3](https://www.dropbox.com/s/6cz3p3q5474jks5/Tell_Your_world.mp3?dl=0)

[前前前世.mp3](https://www.dropbox.com/s/f9la8zauqoykorf/%E5%89%8D%E5%89%8D%E5%89%8D%E4%B8%96.mp3?dl=0)

3音でこれだけ鳴ればいい方では？（ポジティブ  
動画にしてYouTubeにあげようと思ってたのが，思ってるだけで3ヶ月経ったので，Dropboxでお茶を濁す．
