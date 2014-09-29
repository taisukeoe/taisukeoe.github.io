---
layout: post
title: "Akka Transactorsはオワコン"
date: 2014-09-30 01:21:47 +0900
comments: true
categories: scala akka stm
---

{% oembed https://twitter.com/jamie_allen/status/516271313258676226 %}

9/28のAkka Meetupで、AkkaでTransaction処理する話題が出たときに、こんなことをつぶやいたら、[Effective Akka](http://shop.oreilly.com/product/0636920028789.do)著者の[@jamie_allen](https://twitter.com/jamie_allen)さんに「それはもうオワコンだよ」と教えていただいた(確かに調べてみると、[Akka 2.3でdeprecated](http://doc.akka.io/docs/akka/snapshot/project/migration-guide-2.2.x-2.3.x.html))ので記事化してみる。

<!--more-->

##Akka Transactorsとは
[Akka Transactors 公式ドキュメント](http://doc.akka.io/docs/akka/2.0/scala/transactors.html)より抜粋,翻訳。

>Generally, the STM is not needed very often when working with Akka. Some use-cases (that we can think of) are:

>* When you really need composable message flows across many actors updating their internal local state but need them to do that atomically in one big transaction. Might not be often but when you do need this then you are screwed without it.
* When you want to share a datastructure across actors.


>一般的に、Akkaを使用しているときにSTMは殆どの場合必要ないけれども、(我々の考えている)ユースケースが幾つかある:

>* 多数のアクター間の内部のローカルな状態を一つの大きなトランザクション内でアトミックに変更する為に合成可能なメッセージフローが必要なとき。こういうケースは余り頻繁にはないが、**必要になったときTransactorsがないととても困るはずだ**。
* データ構造をアクター間で共有したいとき

「Transactorsが無いととても困るはずだ」と書いておきながら、なんで削除されたのか気になったので聞いてみた。

##Akka Transactorsが削除された理由

* Jamieさん曰く、非決定点?(some indeterminate point)においてSTMはライブロックに陥る可能性があるから。*(STMのライブロックについては[このあたり](http://ja.wikipedia.org/wiki/%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%8A%E3%83%AB%E3%83%A1%E3%83%A2%E3%83%AA#.E4.B8.8D.E9.80.8F.E6.98.8E.E6.80.A7)を参照。)*
* Akka Teamの[@patriknw](https://twitter.com/patriknw)さん曰く、Akka Transactorsは[ScalableでもDistributableでもない](https://groups.google.com/d/msg/akka-user/P1VCkauJXN4/DvQ8J4eNnqUJ)から。
* 上記二つは一応分けたけど、これはクラスター環境だとSTMがライブロックに陥りやすい**ので**、ScalabilityやDistributabilityを損なう、ということな気がする(が間違っていたら教えてください)。

##Akka Transactorsの代替手段

* 複数のActorの内部のローカルな状態をatomicに変更したいときは、short-lived actor(短命のアクター)を実装してその中でトランザクションを書く。
* トランザクションの書き方は、そのshort-lived actorのfailure内にロールバック処理を自前で実装するのが、Jamieさんが[Effective Akka内でも](http://www.slideshare.net/shinolajla/effective-akka-scalaio/8)お勧めしている方法。
* でもshort-lived actor内のlocalな環境に限定すれば[STMも使える](https://groups.google.com/d/msg/akka-user/P1VCkauJXN4/DvQ8J4eNnqUJ)はず…なのだけど、そのことを聞いたら["No, no STM at all"](https://twitter.com/jamie_allen/status/516359268723740672)と強く否定された。localな環境ならSTMはオーバースペックで、ロールバック処理さえあれば十分というだけのことかしら。

##最後に

Effective AkkaのCameo Patternという項で、short-lived actorで複数のActorのをコーディネートする例が書かれているのでご参考までに(でもコーディネートするところだけで、トランザクションに相当するコードはない）。

{% oembed http://www.slideshare.net/shinolajla/effective-akka-scalaio/ %}