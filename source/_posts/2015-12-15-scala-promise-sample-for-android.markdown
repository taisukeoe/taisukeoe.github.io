---
layout: post
title: "Scala標準のPromiseがAndroidで便利だという話"
date: 2015-12-15 23:10:05 +0900
comments: true
categories: scala android
---

この記事は、[Scala Advent Calendar](http://www.adventar.org/calendars/904) 13日目です。

今日は`scala.concurrent.Promise`の話をします。

[Promise - Scala Standard Library 2.11.7 - scala.concurrent.Promise](http://www.scala-lang.org/api/current/index.html#scala.concurrent.Promise)

そもそも`Promise`って使いどころがわかりにくいですよね。他人のコードで使ってるの、ほとんど見たことがありません。

公式ドキュメントではProducer-Consumerパターンでの使い方を解説していますが、現実にこの使い方が必要になるケースってあまり遭遇せず、たいていの場合はFuture同士のflatMapによる合成で事足りてしまうと思います。

[Future と Promise - Scala Documentation](http://docs.scala-lang.org/ja/overviews/core/futures.html)

ところが、実はAndroidアプリ開発では頻繁に遭遇するあのパターンが、`Promise`を使うと非常に取り回しが良くなりますので、紹介したいと思います。

<!--more-->

それは、`Intent`で外部`Activity`から何かしらの値を取得し、`onActivityResult`でその値を受け取るパターンです。

[Intent | Android Developers](https://developer.android.com/intl/ja/reference/android/content/Intent.html)

[Activity#onActivityResult | Android Developers](http://developer.android.com/intl/ja/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)

Androidでは、アプリ外部との連携を`Intent`という仕組みを使って制御しています。

取得したい情報や実行したい処理(Action)を`Intent`にセットし、`startActivityForResult`メソッドに渡すことで、外部アプリなどを起動し必要なデータを取得させたうえで、自分のアプリに返ってくる（`onActivityResult`メソッドが呼ばれ、引数にデータが渡される)ことが出来ます。

この`Intent`は大変便利な仕組みですが、その一方で***一連のパイプラインの記述が`startActivityForResult`と`onActivityResult`の間で分断されてしまい、データの流れが追いにくい***コードになっています。

```scala Promise無しの例(パイプラインが分断される)
class MyActivity extends Activity with TypedFindView{
lazy val IMAGE_FETCH_ID = 12345

override def onCreate(bundle: Bundle) {
   //Activity初期化処理など
   //...
   findView(TR.button_image).onClick{
     _ =>
      val intent = new Intent(Intent.ACTION_PICK, MediaStore.Images.Media.EXTERNAL_CONTENT_URI)
      intent.setType("image/*")
      startActivityForResult(intent, IMAGE_FETCH_ID)
      //データパイプラインがここで分断
    }
 }

 override def onActivityResult(requestCode: Int, resultCode: Int, data: Intent): Unit = {
  requestCode match {
    case IMAGE_FETCH_ID =>
    //データパイプラインがここから継続
    if (resultCode == Activity.RESULT_OK)
      findView(TR.image).setImageURI(data.getData)
     else{
       //failed. Do something if needed.  
     }
    case _ => //do nothing
  }
}
}
```

これを`Promise`を使って書き換えてみましょう。

**`Promise`によって、`startActivityForResult`から `onActivityResult`までの流れを、単一`Future`インスタンスの中に閉じ込めたかのように扱うことが出来ます。**

これでパイプラインの記述をスッキリ書くことが出来るようになりました。

```scala Promise有りの例
class MainActivity extends Activity with TypedFindView with ImageLoadable{
lazy val uiContext = UIContext(this)
 override def onCreate(bundle: Bundle) {
   //Activity初期化処理など
   //...

   //データパイプラインを分断させず、そのまま記述できる
   findView(TR.button_image).onClick{
    _ => chooseImageUri().foreach{
      bmp => findView(TR.image).setImageURI(bmp)
    }(uiContext)
   }
 }
}
```

```scala
trait ImageLoadable extends Activity {
  lazy val IMAGE_FETCH_ID = 12345
  private var promise: Option[Promise[Uri]] = None

  def chooseImageUri(): Future[Uri] = {
    promise.filterNot(_.isCompleted).foreach(_.failure(new InterruptedException("Asked to load another image. Aborted.")))
    val p = Promise[Uri]()
    promise = Some(p)
    val intent = new Intent(Intent.ACTION_PICK, MediaStore.Images.Media.EXTERNAL_CONTENT_URI)
    intent.setType("image/*")
    startActivityForResult(intent, IMAGE_FETCH_ID)
    p.future
  }

  override def onActivityResult(requestCode: Int, resultCode: Int, data: Intent): Unit = {
    super.onActivityResult(requestCode,resultCode,data)
    requestCode match {
      case IMAGE_FETCH_ID => if (resultCode == Activity.RESULT_OK)
        promise.foreach(_.success(data.getData))
      else
        promise.foreach(_.failure(ImageNotAvailableException("Failed to fetch image.")))
      case _ => //do nothing
    }
  }
}
```

ビルド可能なサンプルソースコードはこちらです。

[taisukeoe/ScalaPromiseDemo](https://github.com/taisukeoe/ScalaPromiseDemo)

この例と似た、より一般的な例が以下の`Promise`を使ったCallback APIの`Future`化です。
しかしCallback APIの`Future`化は`scalaz.concurrent.Task.async`でも実現できますが、上記の例は実現できません（`onActivityResult`のせい）。

[ScalaFPEvent - ScalaStdFutureExmaple](https://github.com/taisukeoe/ScalaFPEvent/blob/aa6f784353e8e9147fd47ff6303407bd6faf345c/src/main/scala/ScalaStdFutureExample.scala)

`Promise`って意外と使えるジャン、と思ってもらえれば幸いです。

それでは。
