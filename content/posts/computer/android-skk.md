---
title: "多言語改変版 SKK for Android"
date: 2020-04-22T15:18:43+09:00
tags: [
  "computer",
  "android",
]
toc : true
---

[海月玲二氏のSKK for Android](http://ray-mizuki.la.coocan.jp/software/skk_jp.html) を,
多言語で入力できるように改変した.

GitHub - [ha2zakura/android-skk](https://github.com/ha2zakura/android-skk)

## SKKとは

> SKK（エスケイケイ、Simple Kana to Kanji conversion program）とは、オープンソースで開発されている日本語入力システム（日本語入力メソッド）である。

[SKKとは - ニコニコ大百科](https://dic.nicovideo.jp/a/skk)

IMEの一つ.
普通の日本語IMEだと入力した語を逐一変換する必要があるが,
SKKは変換する語を明示的に入力することで,
入力時のストレスが軽減される便利なIME.

詳しい説明は上の大百科がわかりやすい.

> そして慣れたら最後、他の日本語入力システムが微妙に感じるようになり、SKKが入っていなければ文章を打つのが鬱になってくるのである。なんとも罪深い。

その通りである.

## SKK for Android

大百科にもあるが, SKKをAndroidに移植した猛者がいる.

[Android向けにSKKを作ってみた - minghaiの日記](https://minghai.hatenadiary.org/entry/20090502/p1)

そして, これを基に日本語フリックキーボードを作った猛者がいる.

[SKK for Android (海月玲二氏)](http://ray-mizuki.la.coocan.jp/software/skk_jp.html)

SKKの機能をのままにフリックキーボードが使え,
またBluetoothキーボードを接続した際に普段通りのSKK入力ができる.
これが大変便利である.

### なぜ改変するか

しかし, Android標準のキーボードである **Gboard** は,
標準が故に多言語での入力を容易に実現できる.
趣味柄, 特殊文字を入力したい場面も多く,
その度にキーボードを切り替えるのが面倒でいつしかSKKから離れていた.

しかし, GboardにもSKKができない以外の欠点がある.
Bluetoothキーボードだと日本語⇔英字の切り替えが上手くできない.
これは極めて重大な問題である.

そこで, SKK for Androidを改変することによって
自分好みのキーボードに仕上げてしまおうと考えた.

---

## 改変内容

[リポジトリのREADME](https://github.com/ha2zakura/android-skk/blob/master/README.md)にも書いたのでそちらも参照されたい.

### 日本語入力

{{< figure class="right" src="/img/posts/computer/android-skk-11.jpg" title="日本語入力" >}}

日本語のキーボードは基本的にはそのままであるが,
いくつかの機能を追加した.

  1. **`声`キー, マッシュルームキーの削除**  
 
  音声入力は残してもよかったかもしれないが,
  使わないので削除.
  マッシュルーム機能はもう使わない.
  ただし, これらはレイアウト(`app/src/main/res/xml/keys_flick_jp.xml`)から削除しただけだから,
  本質的には無効化されていない.
  権限を完全に削除するためには`.kt`ファイルを弄る必要があると思われる.

  2. **`◀`,`▶`キーをフリックすることで上下左右に移動する**
  3. **`, . ? !`キーを下フリックすることで`...`を入力できる**
  4. **`や`キーを左右にフリックすることで`(`,`)`を入力できる**
  5. **`わ`キーを下フリックすることで`~`を入力できる**
  
  これらはGboardにあった機能で, 便利だったので実装.

  6. **スペースキーを左フリックすることでシフトができる**

  SandS風便利機能として実装.
  大型画面で左上のシフトをいちいち押すのは(慣れたが)面倒臭かった.
  シフトの反転は以下のようにする.

  ```kotlin
  isShifted = !isShifted
  onSetShifted(isShifted)
  ```

  7. **`小`キーを上フリックすることで`w`を入力できる**
  8. **ポップアップ, 変換候補を黒基調のデザインに変更**

  これらは個人的な好みである.
  設定で切り替えできるようにしたかったが,
  そこまでやる気にならなかった.

  ちなみにフリックごとの条件分岐は
  `app/src/main/java/(domain)/skk/FlickJPKeyboardView.kt`の
  `release()`で

  ```kotlin
  when (mLastPressedKey) {
    // 左矢印キーのとき
    KEYCODE_FLICK_JP_LEFT  -> when (mFlickState) {
      // 右にフリックされたとき
      FLICK_STATE_RIGHT -> if (!mService.handleDpad(KeyEvent.KEYCODE_DPAD_RIGHT)) {
        // 右矢印キーが押されたことにする
        mService.keyDownUp(KeyEvent.KEYCODE_DPAD_RIGHT)
      }
      // ...
    }
    // ...
  }
  ```

  `when()`構文で実装されている.
  Kotlinを使うのはこれが始めてだったが,
  この構文はとても高級で良いと思った.

### 外国語入力

ここからが趣味であるが,
多言語改変版では5つの外国語キーボードを持つ.

  1. **ギリシア文字キーボード**
  2. **エスペラントキーボード**
  3. **ラテン語キーボード**
  4. **ドイツ語キーボード**
  5. **ロシア語キーボード**

ヨーロッパの主要言語を網羅するために,
語派を意識しつつ分類した.
これらはスペースキーを左右にフリックすることで切り替えられる.
また, 一度日本語に戻った後にもキーボードが保持されるのもポイントである.
(日本語 ⇔ ドイツ語 と交互に入力ができる. )

これらの処理のために,
`app/src/main/java/(domain)/skk/QwertyKeyboardView.kt`に
フリック処理を追加した.
と言っても,
ただ`FlickJPKeyboardView.kt`から関係するの関数をコピペするだけであるが.

切り替えの処理は, 新たにキーボードの番号を示す変数`mKeyboardCount`を用意し,
その番号のキーボードを表示させるだけ.

``` kotlin
private fun setKeyboard() {
  mSymbolsShiftedKeyboard.isShifted = false
  when (mKeyboardCount) {
    0 -> keyboard = mGreekKeyboard
    1 -> keyboard = mEsperantoKeyboard
    2 -> keyboard = mLatinKeyboard
    3 -> keyboard = mGermanKeyboard
    4 -> keyboard = mRussianKeyboard
  }
}
```

`m(言語名)Keyboard`は`SKKKeyboard()`によってそれぞれにXMLを適用させている.
このXMLを作るのが大変だった.

以下, 各キーボードの説明をする.
画像のレイアウトの関係で,
Wikipediaで字数稼ぎをしているが読み飛してほしい.

#### 1. ギリシア文字キーボード

{{< figure class="right" src="/img/posts/computer/android-skk-01.jpg" title="ギリシア文字入力" >}}

想定言語 : **現代ギリシア語**, 古代ギリシア語, コプト語

ギリシア語キーボードの需要はさておき,
不定元として使うときにわざわざ予測変換するのが嫌いである.
GeoGebraなどでは元々ギリシア文字キーボードが用意されており,
これが地味に便利であるので作った.

ついでに, ギリシア語から派生したコプト文字のキーボードとしても機能する.

> コプト語（英語: Coptic, コプト語: Ⲙⲉⲧⲣⲉⲙ̀ⲛⲭⲏⲙⲓ met rem en kīmi）もしくはコプト・エジプト語 (Coptic Egyptian) は、4世紀以降のエジプト語をさす用語である。

[コプト語 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%97%E3%83%88%E8%AA%9E)

キーを長押しすると,
画像のようにポップアップが出てきてコプト文字を入力することができる.
コプト文字をどうしても入力したいときに便利だろう.

#### 2. エスペラントキーボード

{{< figure class="right" src="/img/posts/computer/android-skk-02.jpg" title="エスペラント入力" >}}

想定言語 : **エスペラント**, ポーランド語

ギリシア語と比べ, エスペラントの方が日本人には需要があるのではないだろうか, いやないな.

エスペラントの説明はいまさら面倒臭いのでWikipediaより引用.

> エスペラント (Esperanto) とは、ルドヴィコ・ザメンホフとその弟子（協力者）が考案・整備した人工言語。母語の異なる人々の間での意思伝達を目的とする、国際補助語としては最も世界的に認知され、普及の成果を収めた言語となっている[要出典][1]。

[エスペラント - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%82%B9%E3%83%9A%E3%83%A9%E3%83%B3%E3%83%88)

要出典.
そしてこのザメンホフの母国語がポーランド語である.
装飾付きの文字が比較的多いポーランド語であるが,
エスペラントキーボードに追加すると意外とすっきり収まった.
次のラテン語キーボードに入りきらなかったので都合がよい.

#### 3. ラテン語キーボード

{{< figure class="right" src="/img/posts/computer/android-skk-03.jpg" title="ラテン字入力" >}}

想定言語 : **ラテン語**, **英語**, フランス語, スペイン語, イタリア語, ポルトガル語, ラトヴィア語, チェコ語

日本人にとって最も需要があるキーボード.
多言語に対応しているように見せているが
ただ必要な装飾文字が含まれているだけである.
西スラブ語群でポーランド語とチェコ語で分かれてしまっているが,
気にしない.
ラトヴィア語は, 
国際言語学オリンピックのラトヴィア・ヴェンツピルス大会が
来年度に延期されたのに関係する.

ラテン語のキーボードということで,
`·`キー(U+00B7)を付けた.

> ラテン語では、語中の区切りに使われていた。  
> 例) DONA·NOBIS·REQVIEM

[中黒 - Wikipedia](https://ja.wikipedia.org/wiki/%E4%B8%AD%E9%BB%92#%E3%83%A9%E3%83%86%E3%83%B3%E6%96%87%E5%AD%97)

是非, 活用したい.
けどラテン語は無活用がいいな.

あとついでに,
Termux等で使い易くすることも考えて,
一部のキーに記号を割り当てている.


#### 4. ドイツ語キーボード

{{< figure class="right" src="/img/posts/computer/android-skk-04.jpg" title="ドイツ語入力" >}}

多言語版キーボードを作った1番の理由.
[SSK for Androidにウムラウト付き文字を追加した人もいる](https://www.chinpui.org/hack/skk-fk/).
同じことを思う人はやはりいるものだ.

ドイツ語のついでに西・北ゲルマン語群のいくつかを追加した.
**å**, **æ**, **ø**, **á**, **ö** あたりの音価が言語によって違うということを初めて知った. というか, いまいちまだよくわかっていないので, 違ったら指摘してほしい.

あと, ルーン文字も追加した.

> ルーン文字（ルーンもじ）は、ゲルマン人がゲルマン諸語の表記に用いた古い文字体系であり、音素文字の一種である。

[ルーン文字 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%AB%E3%83%BC%E3%83%B3%E6%96%87%E5%AD%97)

アナ雪かなんかで話題になったらしいので,
十分需要はあるだろう.
あるに違いない.

#### 5. ロシア語キーボード

{{< figure class="right" src="/img/posts/computer/android-skk-05.jpg" title="ロシア語入力" >}}

キリル文字キーボードは[海月玲二氏も製作している](http://ray-mizuki.la.coocan.jp/software/skk_jp.html)ので需要は間違いない.
本当はブルガリア語などにも対応したかったが,
キリル文字は多すぎてよくわからなかった.

なので, より需要がありそうなグラゴル文字を追加した.

> グラゴル文字（グラゴルもじ、ロシア語: Глаголица、グラゴーリツァ）は、主にスラヴ系言語を記述するために作られたアルファベットで、スラヴ圏最古の文字である。

[グラゴル文字 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B0%E3%83%A9%E3%82%B4%E3%83%AB%E6%96%87%E5%AD%97)

また, グラゴル文字に対応させるために
一部の初期キリル文字を追加している.
音価が近い文字のキーに追加したはず.
古代教会スラヴ語を常用する人にとっては,
いちいち対応するキーを長押しする必要があるので面倒かもしれない.

古代文字のみのキーボードも面白そうだ.
夢が拡がる.
(需要は...)

---

## インストール

[GitHub Actions](https://github.com/ha2zakura/android-skk/actions)
最新ワークフローのArtifactsよりZIPファイルでダウンロードできる.  
Google Playストアで配信する予定はない.

### 対応端末

動作確認は Huawei P20 (Android 9) で行なっている.
最近のAndroidならば動くんじゃないかな.

### GitHub Actionsによるビルド

[GitHub Actions で Android アプリをビルドして apk ファイルをアップロードする - Qiita](https://qiita.com/hkusu/items/30843c34f569d9a14fef)

このワークフローより,
Lint, Testを省略して導入した.
GitHub Actionsは`git push`するだけでビルドできるので便利.
スマホからでも編集できるので,
このキーボードの使い処ではないだろうか.

## 課題

Android用キーボードの作り方がまだいまいちわからない.
デザインをもっとモダンにしたいができなかった.

あとまだバグがある.
コードを理解しきれてないからでもあるが,
時間があれば0から作り直すべきかもしれない.
