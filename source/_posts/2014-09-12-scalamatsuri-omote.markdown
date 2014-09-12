---
layout: post
title: "ScalaMatsuri 2014 OMOTE/表"
date: 2014-09-12 10:33
comments: true
categories: scala scalamatsuri
---

{% img /images/20140912/scala_matsuri_A.png 600 450 %}

9/6,7の2日間、[ScalaMatsuri 2014](http://scalamatsuri.org/)という日本最大（おそらくAsiaでも最大?)のScalaのカンファンレンスを開催し、総来場者数が400人強と盛会のうちに幕を閉じました。
ご来場者、スポンサー企業、ニコ生視聴者、スタッフの皆様、本当にありがとうございました。

今回は、主にOMOTE / 表側の紹介です。写真なども後日この記事に追加予定。裏側紹介はアンケートデータの整理などが終わり次第、また書きます。
<!--more-->

#1日目:カンファレンス
[@okapie](https://twitter.com/okapie)さんによるtogetterは[こちら](http://togetter.com/li/717717)。

##会場A
###■基調講演 - Scala進化論
Scala作者の[Martin Odersky教授 @odersky](http://twitter.com/odersky)による、Pizza, FunnelといったScalaの前身の言語の紹介からDot計算の話まで含めた、Scalaの歴史と展望の紹介。Scalaが生まれてからの時間が、ちゃんとScalaMatsuriの日に合わせてあるという凝りっぷり、とは[@qtamaki](https://twitter.com/qtamaki)さんの談。

[ScalaはPizzaで作られていた？](http://d.hatena.ne.jp/xuwei/20140911/1410398113)という[@xuwei_k](https://twitter.com/xuwei_k)さんの記事もご覧になると、より楽しめるかも。

{% oembed http://www.slideshare.net/scalaconfjp/the-evolution-of-scala-scala %}

###■sbt,傾向と対策
sbtコミッタで、最近はほぼ一人？でsbtのコードを書いている[Eugene Yokota氏 @eed3si9n_ja](https://twitter.com/eed3si9n_ja)のセッション。昨年のカンファレンスから、ずっと翻訳チームリーダーとしても活躍されています。

今回はmkdirから始まるライブコーディングや、sbt server,　Auto Plugins, Consolidated resolutionといった新機能の紹介まで盛りだくさんなセッション。

{% oembed http://www.slideshare.net/scalaconfjp/sbt-past-and-future-sbt-39003374 %}

###■Fifty Rapture One-Liners in Forty Minutes
[Cake Patternの名付け親](http://okapies.hateblo.jp/entry/2013/07/15/232456)でもある[Jon Pretty氏 @propensive](https://twitter.com/propensive)によるセッション。[Rapture I/O](http://rapture.io/)の1行コードを、50個一気に紹介するという構成でした。1行でパワフルなI/O操作ができるRapture I/Oの魅せ方として、大変上手いなと感心していました。
HTTPやFileのI/Oも簡単で魅力的ですが、JSON周り、特にString InterporlationでExtractor作れるあたりはすごく面白いなと。

```scala
vote match {case json"""{"vote":{"name":$c}}""" => "Voted for"+c.as[String]}
  
// res:String = "Voted for Barak Obama"
```

また2日目のアンカンファレンスではRapture I/OのWorkshopも開催してくださいました。海外からJon Prettyさんのような方がもっと来やすくなるような仕組みを作って行きたい。

###■Node.js vs Play Framework
LinkedInでPlayチームのリーダーを務めていたYevgeniy(Jim) Brikman氏。
LinkedIn内で行っているWeb Framework比較結果の、一番良いところだけ見せていただいているという大変贅沢なセッション。網羅的で内容も大変参考になりますし、翻訳に先んじて日本人聴衆の笑いをかっさらっていく話の上手さ。後述しますが色んな意味でイケメン。

{% oembed http://www.slideshare.net/brikis98/nodejs-vs-play-framework-with-japanese-subtitles %}

あと築地の寿司が大変美味しかったそうで、感激されていました。

{% oembed　https://twitter.com/brikis98/status/507372251369320448 %}

{% oembed　https://twitter.com/brikis98/status/507421447006855168 %}

###■Building a Unified "BigData" Pipeline in ApacheSpark
Apache SparkコミッタのAaron Davidson氏による、Sparkの解説とLive Demo。

実はこのとき直前の映像関係のトラブルでAaron Davidson氏のPCがプロジェクタに映らず、急遽私のPCで発表からLive Demoまでやる(しかもハイクオリティ)という離れ業をやってのけました。昨年の[James Roper氏のデモ](http://scalamatsuri.org/2013/ja/program/index.html#A16402)といい、どうもScalaMatsuriのゲストはLive Demoが抜群に上手い傾向にあるようです。でもJISキーボードはだいぶ苦戦されてました。プレゼン中、私が隣に座って「"_"はどこ？」「ここだよ」みたいなことをずっとやっていたり。

皆さんの声を聞いていると、Databricks Cloudのvisualization機能に驚いた方も多かったようです。このサービスは現在closedβで、価格は未定なもののフリーミアムモデルを検討されているとか。

{% oembed http://www.slideshare.net/scalaconfjp/building-a-unified-data-pipline-in-spark %}

###■Getting started with Scalding, Storm and Summingbird
Twitter Inc.にお務めの[丹羽さん @niwさん](https://twitter.com/niw)が、はるばる日本までいらしてくださいました。

バルス! => TPS(tweets per second)がやばい => じゃあどうやって測るの？

という風に、ネタスライドからScalding,Storm,Summingbirdに繋げる話の上手さ。

{% oembed　https://speakerdeck.com/niw/getting-started-with-scalding-storm-and-summingbird %}

あと[@xuwei_k](https://twitter.com/xuwei_k)さんがこんなことを仰ってるぐらい、講演以外のトークも面白かったです。ありがとうございました。

{% oembed https://twitter.com/xuwei_k/status/508665375530049536 %}

あと、SwiftとScalaの話は何かの機会にぜひ聞きたい(やりたい)です。[Swiftで、Scalaのパターンマッチの文法を実装する話](http://www.slideshare.net/lewuathe/swift-seminar2)とか楽しい。

###■Scala 上で実現された制約プログラミングシステム Scarab について
SAT型制約プログラミングシステムScarabのお話。ドキュメントは[こちら](http://kix.istc.kobe-u.ac.jp/~soh/scarab/)にまとまっています。

私は不勉強ゆえ内容をよく理解できなかったのですが、どうやらsbt内でもSATを利用しているらしく、sbtコミッタのYokotaさんと熱く語ってらっしゃるのを隣でみていました。

宋さんのようなアカデミアにいらっしゃる方が、ScalaMatsuriのCFPに応募してくださって嬉しいですし、アカデミアの方が市井のScalaのコアな開発者と出会う機会をもっと作りたい。

{% oembed http://www.slideshare.net/scalaconfjp/scalamatsuri-sohscaraben %}

###■懇親会
実は、この前日の9/5がOdersky教授の56歳の誕生日。というわけで、サプライズで歌ってケーキをプレゼントしました。すごく喜んでいただけて良かった。

このアイディアをあの忙しい中考えついた[@takezoux2]((http://twitter.com/takezoux2)さんの幹事力の高さが半端ない。

{% oembed https://twitter.com/propensive/status/508216380785557504 %}

#2日目:アンカンファレンス
アンカンファレンスは、参加者がその場で聞きたいセッション、話したいセッションのアイディアを出し合いながら、その場で作って行くイベントです。今回初の試みでしたが、大いに盛り上がり、本当に良かった(内心結構心配でした)。。。

[@okapie](https://twitter.com/okapie)さんによるtogetterは[こちら](http://togetter.com/li/717660)。

###■DDDの話
[@j5ik2o](https://twitter.com/j5ik2o)さんによる、DDD(~~Diet~~ Domain Driven Design)の解説。比喩から具体例まで、とても分かりやすく大変好評でした。

###■Akkaの話
[@xuwei_k](https://twitter.com/xuwei_k)さんによる、Akkaの初心者向けセッション。
そういえばTypesafe社のAkkaコミッタが今月日本にいらっしゃるので、[Akka Meetup](http://connpass.com/event/8622/)がドワンゴさんで9/28に開催されるとか。

###■ランチ
最近転職した人の話を聞いた後、今すぐ転職したい人！と[@kiris](https://twitter.com/kiris)さんが聞いたら、数十人が一気に手をあげて大いに盛り上がり、運営側が逆に驚くくらいでした(@kirisさん、ナイス！)。ScalaMatsuri 2014が縁となって転職した方、ブログ待ってます！

{% oembed https://twitter.com/scala_jp/status/509229988587896832 %}

###■Scala --the simple parts--
Odersky教授によるセッション2個目。一時期Twitterでも話題になった、CanBuildFromの話を直接聞けたのが私としては良かった。

{% oembed http://www.slideshare.net/Odersky/scala-the-simple-parts %}

###■Code clinic by guest speakers
Odersky教授を始めとしたゲストに、マサカリを直接投げてもらえるセッション。
ランチでぽろっと出たアイディアなのですが、Odersky教授が超乗り気で非常に盛り上がりました。

フィードバックの抜粋

* 不要な括弧は外す

```scala
if(config.isUser){
	println("He is a user!")	
}else{
	println("He is not a user!")		
}
```
より
```scala
if(config.isUser)
	println("He is a user!")	
else
	println("He is not a user!")		
```

* コメントは「何をしているか」ではなく「なぜそうなったのかという経緯」を書くべき
* メソッドのドットと括弧を外すかどうかについて、例えば

```scala
hoge.startsWith("h")
```
より
```scala
hoge startsWith "h"
```
という風に英語風に書いた方がScalaっぽいと言われる([去年のJoshua Suereth氏のセッション](http://www.slideshare.net/scalaconfjp/coding-in-style)など参照)けど、最近はOdersky教授自身はその判断に迷う事が多くなったそう。実際最近Odersky教授のも、ドットと括弧を付けて書いたりしているらしい。

但し
```scala
numList map {_ * 5 + 1} filter {_ > 13}
```
みたいなのはダメ。ドットと括弧を使った方が確実に見やすい、との談。

そして実はこのセッション、ゲストのYevgeniy Brikman氏によるアイディア。でもそれを紹介しようとしたら「自分のアイディアだっていうとバイアスがかかるから」といって固辞されたりで、本当にイケメン。

{% oembed https://twitter.com/brikis98/status/509087498438340608 %}

ScalaMatsuri発のセッションとして、世界に広まってくれると嬉しいですね。

###■パネルディスカッション
大変盛り上がっていたようですが、他の仕事をしていて聞けなかった('；ω；`)ﾌﾞﾜｯ ので@okapieさんのtogetterの[このへん](http://togetter.com/li/717660?page=12)をご参照ください。

#総括
昨年に比べて発表内容が非常に幅広くなり、日本でScalaの普及が進んでいる事をまざまざと感じさせるイベントとなりました。また予想を上回る数のスポンサー企業さまにお申し込みをいただきまして、逆にスポンサーLT枠が枯渇するというほどの盛況ぶりでした。特典枯渇のためお断りしてしまったスポンサー企業のみなさま、大変申し訳ありませんでした。もしよろしければ、また次回お声がけください。

あと、海外ゲストや一般参加者、スポンサー企業の多くの方から、日本のScalaコミュティが熱い & 勢いありますね！というお声を沢山、本当に沢山かけていただき、イベント運営を担っている身としては非常に嬉しく感じております。皆さんのお陰です。これからも一緒に盛り上げて行きましょう！

その一方で、初心者向けセッションが手薄になってしまったのは反省点。翻訳が足りない、という声も真摯に受け止めており、来年は同時音声通訳の導入を真剣に検討しております。その辺りはまた次回のエントリにて。