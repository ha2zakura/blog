---
title : "PIC16F84A : マイコンボード作リ"
tags : [
  "electronics",
  "pic16f84a",
]
date : "2018-01-11T16:11:35+09:00"
toc : true
---

去年の夏に諸事情でPICマイコンを触る機会があって,   
PICマイコン面白いな, と興味を持ち,   
本格的にPICを勉強したいなと思った.   
<!--more-->
なのでPICを始めるべく色々揃えた.

## マイコンボードを作る

いちいちブレッドボードで回路を組むのは嫌いなので,   
例の如くまたマイコンボードを作った.   

### PIC16F84A本体

{{< figure src="/img/posts/pic16f84a/board-1.jpg" title="PIC16F84A" >}}

有名なPICマイコンである, 16F84Aを選んた.   

[ＰＩＣマイコン　ＰＩＣ１６Ｆ８４Ａ－２０Ｉ／Ｐ: マイコン関連 秋月電子通商-電子部品・ネット通販](http://akizukidenshi.com/catalog/g/gI-00097/)

PICはDIPの選択肢がいっぱいあっていい.   
メモリ増量版である[16F88](http://akizukidenshi.com/catalog/g/gI-00567/)もあるが,   
まずは16F84Aから勉強していきたいと思う.

### PICkit3

PICマイコンを書き込むには特別な書き込み機が必要.   
知識があれば自分でも作れるそうだが,   
初心者の私には難しいので純正ライタ「PICkit3」を買った.


[マイクロチップ　ＰＩＣｋｉｔ３: マイコン関連 秋月電子通商-電子部品・ネット通販](http://akizukidenshi.com/catalog/g/gM-03608)



約6,000円. この値段で知識と安心を得られるのなら安い, はず.

### マイコンボード

{{< figure src="/img/posts/pic16f84a/board-2.jpg" title="PIC16F84Aマイコンボード" >}}

去年の3月に発売されたちっちゃいユニバーサル基板にまとめてみた.

[小型ユニバーサル基板　４５×４５ｍｍ: パーツ一般 秋月電子通商 電子部品 ネット通販](http://akizukidenshi.com/catalog/g/gP-11735/)

{{< figure src="/img/posts/pic16f84a/board-3.png" title="" >}}

左のピンヘッダはPICkit3との接続用.   

左下の[マイクロBメスUSBのコネクタ](http://akizukidenshi.com/catalog/g/gK-06656/)は給電（5V）用.   
作った後で知ったのが, [こっち](http://akizukidenshi.com/catalog/g/gK-10972)の方が安い.   

右下のLEDは左からRB0, RB1, RB2につながっている. 動作確認用.   
USBコネクタの左のLEDは電源確認用.

上のセラミック発振子は4MHZにした.  

{{< figure src="/img/posts/pic16f84a/board-4.jpg" title="裏面" >}}

 1度配線を間違えたせいでヤニだらけ.   
今度からちゃんと確認しよ...

{{< figure src="/img/posts/pic16f84a/board-5.png" title="回路図" >}}

PICkit3はCN1のピン1を三角形と合うように接続する.

{{< figure src="/img/posts/pic16f84a/board-6.jpg" title="PICkit3との接続" >}}
