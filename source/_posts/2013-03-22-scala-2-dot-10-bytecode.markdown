---
layout: post
title: "Scala2.10.x bytecode problem"
date: 2013-03-22 20:32
comments: true
categories: scala jvm
---
# 概要
Scala 2.10以上でAndroid4.1以上のライブラリと一緒にコンパイルしたプロジェクトを、Android4.0以下の端末で動作させるとNoSuchMethodErrorがthrowされます。原因はScala2.10.x系のコンパイラの不具合。親クラスで実装されたメソッドをそのまま継承した場合、**2.9.x及びJavaではメソッドのqualifying typeの参照が子クラス**になるのに対し、**2.10.xではqualifying typeの参照が親クラス**になってしまうことが直接の原因です。

詳しくは第99回[rpscala](http://www.scala-users.org/shibuya/)で発表したスライドをどうぞ。

{% oembed http://www.slideshare.net/oeuia/scala210x-bytecode-problems-in-android %}

このバグと同種のものがJIRAでも見つからなかったので、issue報告を上げた結果、3/23に無事修正されたようです。

[[#SI-7253] SQLiteDatabase cannot be closed in Android 4.0 or below devices, in apps built by Scala 2.10.x w/ Android 4.1 due to Scala 2.10.x bytecode problem. - Scala](https://issues.scala-lang.org/browse/SI-7253)

rpscalaでの発表時によくわからなかった以下のポイントがクリアになったので、フォローアップとして記載しておきます。

* これは意図された仕様変更なのか？
* これはバグなのか？
* いつ直るのか？	

<!-- more -->


#1. これは意図された仕様変更なのか？
特段意図されたものではなく、Scala2.10でコンパイラをfjbgからasmへ変更した際に、紛れ込んだ模様です。
(該当commitのauthorのPaul Phillipsさんが"I don't think this is asm's doing, but not sure of that either.(ASMのせいじゃないように思うけど、どうだか分からないな。)"と発言している。)

[Fix for erroneous bytecode generation. · 0bea2ab · scala/scala](https://github.com/scala/scala/commit/0bea2ab5f6)


#2. これはバグなのか？
バグのようです。根拠として引用された、Java Language Specification(以下JLS)内の[Binary Compatibility](http://docs.oracle.com/javase/specs/jls/se7/html/jls-13.html)に関する記述は以下の通り。

>* "**Moving a method upward in the class hierarchy.**"(メソッドがクラス構成の上流に移動することについては、バイナリ互換性を保たなければいけない)"	
		
>* "A reference to a method must be resolved at compile time to **a symbolic reference to the erasure (§4.6) of the qualifying type of the invocation**, plus the erasure of the signature of the method (§8.4.2)."(メソッド又はコンストラクタへの参照は，そのqualifying typeへの記号参照に，そのメソッド又はコンストラクタのシグネチャを加えて，コンパイル時に解決しなければならない。)
	
>*	If the expression is of the form **Primary.f** then:
	
>		If the compile-time type of Primary is an intersection type (§4.9) V1 & ... & Vn, then the qualifying type of the reference is V1.

>		Otherwise, **the compile-time type of Primary is the qualifying type of the reference.**" 

>		(Pimary.fという表現があったとき、Primaryがintersectional type以外の場合、Primaryがqualifying typeとなる。)

それに対して、以下のような反論がありました（が、反論の発言主が"You got me, it's a bug."(やっぱりあれはバグだったよ。)と訂正済み。）

* JLSはbackward compatibility(後方互換性)について定めたもので、今回のケースのようなforward compatibility(前方互換性)については当てはまらない。
* 異なるバイトコードに大してコンパイルした場合、異なるものが生成されるのは当たり前である。

そもそもScalaがJLSに準拠すると明確に宣言しているソースを見つけられなかったのが個人的には気になっていましたが、そこは争点になりませんでした。
*(注:というのも、以前Scala作者のMartin Odersky教授が別のissue [[#SI-1806] Can't access protected static inner classes of extended classes](https://issues.scala-lang.org/browse/SI-1806))] で **"I disagree. We can never achieve 100% java interop without becoming Java."(私は反対だ。Javaとの相互運用を100%達成するには、Java自体になる他ない。)**と発言していたので。)*
とはいえ、僕が見落としているだけのような気がするので、もしその辺りご存知の方がいたら教えてください。


#3. いつの直るのか？

3/23に、本バグ修正のpull request([SI-7253: respect binary compatibility constraints by Blaisorblade](https://github.com/scala/scala/pull/2264))がmergeされましたので、おそらく2.10.2で反映されるものと思われます。

ここに関して、2.10系のbinary compatibilityを破壊するから2.11まではXfutureオプションに入れておくべきでは？という議論がありましたが、James Iryさん（[Java to Scala with the Help of Experts | The Scala Programming Language](http://www.scala-lang.org/node/960)などを書いてる方）の意見が通ったようです。

>This patch will improve binary compatibility when source changes in the way that happened in SI-7253. That improvement won't be available in prior 2.10.x versions so in that sense this commit is a change in behavior. But since the old behavior was to create a linkage error I'm okay with that change.

>(このパッチは、SI-7253のようなソースコードの変更において、バイナリ互換性を改善します。以前の2.10.x versionでは使用不可能という意味で、このパッチはバイナリの振る舞いを変更しますが、以前の振る舞いがリンクエラーを生成してしまう以上、私はこの変更を適用しても構わないと思います。)

2.10.2がリリースされるまでは繰り返しになりますが、**以下のような構成のプロジェクトは予期せぬNoSuchMethodErrorをruntimeでthrowするため、避けるべきです**。

	Android platform version 16以上 (4.1以上に相当)
	Android minSDKversion 15以下 (4.0以下に相当)
	scalaVersion 2.10.x


###おまけ1
[@jsuereth](https://twitter.com/jsuereth/)に褒められたヽ(・∀・)ﾉ ﾜｰｲ

{% oembed https://twitter.com/jsuereth/status/312553768644395008 %}


###おまけ2

バグの再現手順。

再現用のrepositoryはこちら。
[taisukeoe/scala_2_10_android_error · GitHub](https://github.com/taisukeoe/scala_2_10_android_error)

	Android platform version 16以上 (4.1以上に相当)
	Android minSDKversion 15以下 (4.0以下に相当)
	scalaVersion 2.10.x
	
で以下のようなソースコードのプロジェクトを実行すると、NoSuchMethodErrorがthrowされます。

####Project whole souce codes:

``` scala

	class DemoActivity extends Activity{
	  override def onCreate(savedInstanceState: Bundle) {
    	super.onCreate(savedInstanceState)

	    new SQLHelperDemo(this).getReadableDatabase.close()

    	val textView = new TextView(this)
	    textView.setText("Do I still survive?")
    	setContentView(textView)
  		}
	}
	class SQLHelperDemo(context:Context) extends SQLiteOpenHelper(context,"demo",null,1){
	   def onCreate(p1: SQLiteDatabase) {}
	   def onUpgrade(p1: SQLiteDatabase, p2: Int, p3: Int) {}
	}
```	


####Full stack traces:

	W/dalvikvm(7951): threadid=1: thread exiting with uncaught exception (group=0x40abd210)
	E/AndroidRuntime(7951): FATAL EXCEPTION: main
	E/AndroidRuntime(7951): java.lang.NoSuchMethodError: android.database.sqlite.SQLiteClosable.close
	E/AndroidRuntime(7951): 	at com.hemplant.demo.no_such_method_in_2_10.DemoActivity.onCreate(DemoActivity.scala:18)
	E/AndroidRuntime(7951): 	at android.app.Activity.performCreate(Activity.java:4465)
	E/AndroidRuntime(7951): 	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1049)
	E/AndroidRuntime(7951): 	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:1931)
	E/AndroidRuntime(7951): 	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:1992)
	E/AndroidRuntime(7951): 	at android.app.ActivityThread.access$600(ActivityThread.java:127)
	E/AndroidRuntime(7951): 	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1158)
	E/AndroidRuntime(7951): 	at android.os.Handler.dispatchMessage(Handler.java:99)
	E/AndroidRuntime(7951): 	at android.os.Looper.loop(Looper.java:137)
	E/AndroidRuntime(7951): 	at android.app.ActivityThread.main(ActivityThread.java:4441)
	E/AndroidRuntime(7951): 	at java.lang.reflect.Method.invokeNative(Native Method)
	E/AndroidRuntime(7951): 	at java.lang.reflect.Method.invoke(Method.java:511)
	E/AndroidRuntime(7951): 	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:823)
	E/AndroidRuntime(7951): 	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:590)
	E/AndroidRuntime(7951): 	at dalvik.system.NativeStart.main(Native Method)


