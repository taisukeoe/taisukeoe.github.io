---
layout: post
title: "[Deprecated]ScalaでAndroidアプリを作るには(IntelliJ IDEA + sbt)"
date: 2013-06-08 03:25
comments: true
categories: scala android intellij sbt
---

***この記事の内容は執筆当時のものです。最新の内容は[こちら](/blog/2014/03/13/pfn-android-sdk-plugin)を参照してください。***

LinkedInのブログで、EclipseベースのScalaによるAndroidアプリ開発環境が紹介されてました。（と、
[@okapies](https://twitter.com/okapies)さんに教えていただきました。ありがとうございます。)

[The technology behind EatIn: Android apps in Scala, iOS apps, and Play Framework web services](http://engineering.linkedin.com/incubator/technology-behind-eatin-android-apps-scala-ios-apps-and-play-framework-web-services)

うちのAndroid Scala開発環境はIntelliJ IDEA + sbtなので、折角なのでこんな風にも出来ますよと紹介してみます。本当はLinkedInブログに対抗して *(sorry eclipse users!!)* って書きたかったw のですが、IntelliJはエディタとして使ってるだけなので、Eclipseに変えても大して支障ないです。というわけでお好きな方をどぞ。

Androidに馴染みが無い方にも、Scalaに馴染みが無い方にも試してもらえるよう、クドいぐらいに丁寧に書いてますので、慣れ親しんだ部分は読み飛ばしてください。

また、**「もっとこうした方がいいんじゃない？」みたいな突っ込みは大歓迎**です。

#開発環境
以下コマンド例はMac/Linux用なので、Windowsの方はお好きな方法で適宜書き換えてください。

##各種インストール
<!-- more -->
###IntelliJ IDEA
強力なリファクタリングと、Android UI Designer機能が便利なIDE。

[IntelliJ IDEA — The Best Java and Polyglot IDE](http://www.jetbrains.com/idea/)

幸いなことに、IntelliJ IDEAのCommunity Edition(無料版)でもScalaとAndroidのサポートを使えます。
インストールしたら、以下の手順でScalaのPluginを入れましょう。

	Configure => Plugins => Browse repositories 
	プラグインの一覧からscalaを右クリック => Download & Install

ちなみにsbtコンソールをIntelliJに追加するプラグインもありますが、このコンソールは*Tab補完が効かない*ので、コマンドが長めなandroid-pluginには不向き。Terminal経由でsbtコマンドを叩いた方が無難です。



###Android SDK
Android APIのjarが含まれているほか、ビルドやデバッグに使用する各種便利ツールが入ってます。アップデートの度にちょこちょこ構成が変わるので、各種サードパーティ製のツールとの互換性がなくなったりします。なので**新しいバージョンが出ても1個前のバージョンは消さずに、シンボリックリンクの切り替えで対応しとくと良い**です。

[Android SDK | Android Developers](http://developer.android.com/intl/ja/sdk/index.html#ExistingIDE)

IntelliJ IDEAを使う場合は、ADT Bundleではなく*USE AN EXISITING IDE*からSDK単体をDLます。ADT Bundleでも特に問題はないですが、余計な物(Eclipse+ADT)が入ってます。

その後、SDK環境変数ANDROID_HOMEとtools, platform-toolsのディレクトリにPATHを通します。

``` bash
ln -s <SDK_ROOT_DIR> <SYMBOLIC_LINK_PATH>

export $ANDROID_HOME=<SYMBOLIC_LINK_PATH>
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/platform-tools
```
		
ちなみに上記SDKには最新版のAndroid versionのlibraryしか含まれていないので、回線に余裕があるときに、以下のコマンドでAndroid SDK Manager(GUI)を起動し、各versionのlibraryを落としておくと良いです。

``` bash
android
```		

なおSDKにAndroidエミュレーター作成用ツールも含まれていますが、起動と各種動作が重い上に、エミュレーターで試せない機能が多い(カメラなど各種ハードウェア、GCMを使用したpush通知、アプリ内課金)など、不便極まりないので実機でデバッグする方が無難です。
どうしても作りたい(特定のマイナーな解像度での見え方をチェックしたい、など)場合は、以下を参考にどうぞ。

[Emulator - Android エミュレータ - ソフトウェア技術ドキュメントを勝手に翻訳](https://sites.google.com/a/techdoctranslator.com/jp/android/developing/tools/emulator)

###giter8
Githubのリポジトリに登録しておいたテンプレートからプロジェクトを生成するツール。Androidアプリのプロジェクトは色々構成がややこしいので、giter8経由で作成するのがオヌヌメです。

``` bash
brew install giter8
```

[giter8](https://github.com/n8han/giter8)

###scala
Scala本体。まだ入れていなければ。

``` bash
brew install　scala
```
[Scala Documentation](http://docs.scala-lang.org/index.html#)

###sbt
Scalaプロジェクトのビルドによく使われるビルドツール。まだ入れていなければ。

``` bash
brew install　sbt
```

[始める sbt - ようこそ](http://scalajp.github.io/sbt-getting-started-guide-ja/)
			
##使用する各種コマンド
###プロジェクトの作成
giter8を使って作成します。

``` bash
g8 taisukeoe/android-app
		
			Template for Android apps in Scala 

		 	package [my.android.project]: <YOUR_PACKAGE_NAME>
		 	name [My Android Project]: <YOUR_PROJECT_NAME>
		 	main_activity [MainActivity]: 
		 	scala_version [2.10.2]: 
		 	min_api_level [8]: 
		 	useProguard [true]: 
		 	min_api_level [17]: 
				
cd <YOUR_PROJECT_NAME>

git init
		
sbt
gen-idea
```

各変数の解説

* package … Androidアプリのユニーク性の担保に使われているので、絶対にかぶらないように設定する必要があります。同じ端末に同じpackageのアプリは同時に1つしか入れられませんし、GooglePlayには同じパッケージのアプリは常に1つしか存在しません。
* name … プロジェクト名及びディレクトリ名に使われます。お好きに。
* main_activity … アプリ起動時に実行するActivity(画面に相当)の名前。お好きに。
* scala_version … 2.10.2以上推奨です。理由は[こちらのバグ](http://taisukeoe.github.io/blog/2013/03/22/scala-2-dot-10-bytecode/)。2.9系でも別に構いませんが。
* min_api_level … 1がAndroid1.5相当、デフォルトの8はAndroid2.2相当、17が最新のAndroid4.2相当です。このプロジェクトがサポートする最低のAndroid versionの数字を入力します。今使われているAndroid端末の99%程度は2.2以上なので、デフォルトで特に問題ないと思います。
* useProguard … 基本的にtrue一択。あらかじめscala-library.jarをAndroid端末に仕込んだりする場合はfalseでも良いですが。
* target_api_level … 使用したいAndroid APIが含まれている、Android versionを入力します。デフォルトの17(最新4.2)にしておけば全てのAPIを使用できますが、Android4.1以前の端末では該当箇所が実行されないように条件分岐しないと、NoClassDefErrorないしはNoSuchMethodExceptionがthrowされます。もちろんmin_api_levelに揃えておくのも手です。

作成したプロジェクトをIntelliJ IDEAで開くとこんな感じ。なおsbt pluginは[jberkel/android-plugin](https://github.com/jberkel/android-plugin/)及び[mpeltonen/sbt-idea](https://github.com/mpeltonen/sbt-idea)を使用しています。

{% img /images/20130608/template.jpg 600 450 %}

###ビルド定義の更新
sbtのビルド定義ファイル(*project/build.scala*)でAndroid versionを変えたりライブラリ依存性を変更した場合は、忘れずにリロードして再度idea設定ファイルを生成します。でないとIntelliJの補完が効かなくなります。

``` bash
sbt
reload
gen-idea
```

###ビルド&デプロイ
USBデバッグモードをONにしたAndroid端末を接続しておくと、以下のコマンドでビルド及びデプロイが実行できます。

``` bash
sbt
android:start-device
```

ビルドした実行ファイルは*target/<YOUR_PROJECT_NAME>-<VERSION>.apk*に生成されます。
ちなみに、自分以外の誰かにインストールさせたいときに一番簡単な方法は、このapkファイルをメールで送りつけることです。AndroidデフォルトのGmailアプリからapkを開くとインストールできます。

なおAndroidプロジェクトのビルドには時間がかかりすぎる(MBP Retinaで30秒以上...)ので、コード書いてる最中は自動コンパイルだけにしておく方が無難です。

``` bash
sbt
~compile
```

###テスト
Androidアプリのテストは、Android上で行う必要がないもの*(ex. Modelやユーティリティークラスの単体テスト)*と、Android上で行う必要があるもの*(ex. Activity(画面)やViewに関するテスト)*の二通りがあり、実行方法も違います。

Android上では行う必要のないテストは、**src/test/**以下に書いて、以下の通りテストを実行。

``` bash
sbt
test
```

Android上で行う必要のあるテストは、サブプロジェクトとして別途ビルド&デプロイします。ここまで記載した手順通りで作成している場合には、**tests/src/main/**以下にテストコードを書く形になります。実行コマンドはこんな感じです。

``` bash
sbt
project tests
android:install-device
android:test-device
```

余談ですが、[jberkel/android-plugin]((https://github.com/jberkel/android-plugin/)のTypedResourcesを利用したプロジェクトのテストをAndroid上で実行すると、IllegalAccessErrorで失敗するようです。TypedResourcesはただのユーティリティー的なtraitなので、使わなくても支障は特にありません。

###デバッグ
主にAndroidSDK同梱のAndroid Device Monitorを使います。

Logの確認の他、Thread, Heapなどの状態をチェックしたり、スクリーンショットを撮ったり、hprofとったり割と便利です。

``` bash
monitor
```

その他、よく使うandroid-pluginのsbtコマンド一覧

	android:package-debug   … プロジェクトをビルド。
	android:prepare-market   … プロジェクトをビルドし、指定した署名ファイル(<HOGE>.keystore)で署名。GooglePlayへのアップロード時に使用する。
			
	android:install-device   … プロジェクトをビルド、USB経由でAndroid端末にインストール
	android:start-device   … プロジェクトをビルド、USB経由でAndroid端末にインストール、アプリ実行
	android:uninstall-device   … USB経由でAndroid端末からアンインストール
	android:test-device … USB経由でAndroid端末上でテストを実施。前もってテスト用apkをインストールする必要あり。
	android:screenshot-device   …USB経由でAndroid端末のスクリーンショットを撮影
	
	# emulatorの場合は、上記コマンドのdeviceをemulatorに変えてください。		
Wikiにも色々書かれていますので、一読しておくと良いかもです。

[jberkel/android-plugin Wiki](https://github.com/jberkel/android-plugin/wiki)

###補足 Android UI Designer

上記手順だけでも使えないことは無いですが、IntelliJ IDEAのAndroid UI Designer機能を使うためにはもう少し設定が必要です。ちなみに先日のGoogle I/Oで発表された[Android Studio](http://taisukeoe.github.io/blog/2013/05/16/android-studio/)のメイン機能も、このAndroid UI Designerを下敷きにしているぐらい強力だったりするので、折角IntelliJを使うのであれば、ぜひここまで設定しましょう！

以下、IntelliJ IDEAでプロジェクトを開いた後に行います。

	File  => Project Structure => Modules => 
	メインプロジェクトを選択して Add => Android
	
{% img /images/20130608/project_structure_add_sdk.jpg 600 450 %}

追加した"Android"を選択し、Manifest file/ Resources directory/ Assets directoryのパス中の".idea_modules"を全て"src/main"に変更します。Native libs directoryは".idea_modules"を削除しましょう。下記スクリーンショットのような感じです。

{% img /images/20130608/project_structure_path.jpg 600 450 %}

続いてAndroid SDKのパスを追加します。

	File => Project Structure => Platform Settings => SDKs =>
	 上の"+"ボタン => Android SDK => $ANDROID_HOME/platforms/android-17　を選択

	File => Project Structure => Modules => メインプロジェクトのModule SDK => 今追加したAndroid SDKを選択。

{% img /images/20130608/project_structure_project_sdk.jpg 600 450 %}

以上を終えた後、src/main/res/layout/main.xmlを開くと、自動でAndroid UI Designer機能が立ち上がります。もちろん画面下タブのTextを選択すれば、xmlを直接編集できます。Previewを見ながらxmlを編集するだけでも便利です。

{% img /images/20130608/ui_designer.jpg 600 450 %}

##最後に

かなり丁寧に書いたので長くなってしまいましたが、突っ込みor質問orコメントなどあれば下記コメント欄か[@OE_uia](https://twitter.com/oe_uia)までお願いします。