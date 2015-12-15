---
layout: post
title: "ScalikeJDBC on Android SQLite"
date: 2015-05-06 21:40:25 +0900
comments: true
categories: scala android scalikejdbc sqlite 
---

意外にもまだ試している人がいなさそうなので、[rpscala合宿](https://rpscala.doorkeeper.jp/events/23383)でDEMOを作成した。

[scalikejdbc-on-android](https://github.com/taisukeoe/scalikejdbc-on-android)

{% img /images/20150506/scalikejdbc_demo.png %}

<!--more-->

###使用versionなど:

* Scala 2.11.6
* sbt 0.13.8
* Target: Android 5.1 
* MinSDK: Android 4.0.3
* pfn/android-sdk-plugin 1.3.22
* ScalikeJDBC 2.2.6
* SQLDroid 1.0.3

##Android in ScalaのDBアクセス事情
* 自分は普段Android Database API + [RichCursor](https://github.com/pocorall/scaloid/blob/060fd9b5d330d735be96ac0b9489b4600a6dec09/scaloid-common/src/main/st/org/scaloid/common/implicits.scala#L110-L120)を使う
* [Slickを使用したTypesafe activatorテンプレート](https://www.typesafe.com/activator/template/agile-scala-android-example)などもある
* そもそもDBをローカルに持つ必要のない（DBサーバー+クライアントキャッシュな)Androidアプリも多い

##[ScalikeJDBC](http://scalikejdbc.org/)とは？
`a tidy SQL-based access library for Scala Developers`(公式ドキュメントより抜粋)

SQL文をそのまま(より型安全な方法で）扱えるのが特長。Androidでいえば[SQLiteDatabase.rawQuery](http://developer.android.com/intl/ja/reference/android/database/sqlite/SQLiteDatabase.html)を好んで使う人向け。

##SQLite JDBC Driver
今回はAndroidのSQLiteを使用するので、SQLite3に対応していることが必須。あとはPure Java実装か、Android NDKでAndroid用のNative Libraryを生成できる必要がある。

###[SQLDroid](https://github.com/SQLDroid/SQLDroid)
今回使用した、Android Database APIをラップしたJDBC Driver。ただあまり活発にメンテされていないようなので、プロダクションで使用するのは躊躇する。

###[Android JDBC Driver (Oracle)](http://docs.oracle.com/cd/E17076_02/html/installation/build_android_jdbc.html)
Android NDKを使用しビルドすることで、Android用のNative Library(.so)を生成できる…が、unmanagedDependenciesとして追加する必要があるので、ちょっと扱いずらい。（なお、[この記事](http://d.hatena.ne.jp/esmasui/20120918/1347985333)を読む限り、Natvie Libraryを含めなくても動作する（＝Pure Java実装に自動的に切り替わっている？）模様。)

###[xerial/sqlite-jdbc](https://bitbucket.org/xerial/sqlite-jdbc)
現在Androidはサポートされておらず、実行すると

> java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader[DexPathList[[zip file "/system/framework/android.test.runner.jar", zip file "/data/app/taisukeoe.scalikeroid-1/base.apk"],nativeLibraryDirectories=[/vendor/lib, /system/lib]]] couldn't find "libsqlitejdbc.so"

というRuntimeエラーとともに落ちる。

[Pure Java実装に切り替えるAPI](https://bitbucket.org/xerial/sqlite-jdbc/issue/159/javafx-android-port-crashes-and-fails-to)のfeatureリクエストは上がっているようなので、これが実装されれば使えるはず。

##落とし穴|Pitfalls
###[SQLite](https://www.sqlite.org/datatype3.html)起因のもの
* `serial`データ型が存在しないので、`integer autoincrement`にする。
* `timestmap`および`datetime`データ型が存在せず、`timestamp`指定すると`yyyy-MM-dd hh:mm:ss`というISO8601フォーマットのTEXT型で保存されるため、WrappedResultSet.jodaDateTimeでDateTime型の値を抽出できない。`integer`型にしておいてWrappedResultSet.timestampを使う。

###Android起因のもの
* `DatabseUtils.createDbFromSqlStatements`から手動でDatabaseを作成する必要がある

###Android in Scala起因のもの
* [pfn/android-sdk-plugin](https://github.com/pfn/android-sdk-plugin)のProguard後のclassファイルのキャッシュが重複してしまうことにより、ビルド時に以下のExceptionが投げられる

> java.lang.IllegalArgumentException: already added: Lscala/util/parsing/combinator/JavaTokenParsers$class;

これを防ぐために、sbtビルド定義に以下を追加する。

`proguardCache in Android += ProguardCache("parser-combinators") % "org.scala-lang.modules" %% "scala-parser-combinators"`

##雑感
ハッカソンではDEMOまでしか作成できなかったが

* ScalikeJDBCの主だった機能は一通り試したい
* [h2 database](http://www.h2database.com/html/tutorial.html#android)は公式でAndroid対応のJDBC driverを配布しているようなので、h2でも試したい
* Typesafe activator templateにしたい