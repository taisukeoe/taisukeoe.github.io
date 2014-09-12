---
layout: post
title: "Android Studioを触ってみた"
date: 2013-05-16 15:09
comments: true
categories: android ide
---

#概要

[Google I/O 2013](https://developers.google.com/events/io/) 初日(日本時間5/16 1:00)にて発表されました。[IntelliJ IDEA](http://www.jetbrains.com/idea/)ベースのAndroidに特化したオープンソースのIDEだそうです。

そのほかの、Google I/O関連のまとめはこちら。Galaxy S4のNexus化モデルの販売やら、Google Mapの強化、PlatformとしてのChrome、GmailとGoogle Walletの統合など、他にも盛りだくさんな発表がありました。

これら発表を受けて、Googleの株価は過去1年間で最高の915ドルまで急騰した模様。

[Google、Android向け統合開発環境「Android Studio」を発表 - ITmedia Mobile](http://www.itmedia.co.jp/mobile/articles/1305/16/news051.html)

[Google I/O 2013 開幕、初日キーノート速報 - Engadget Japanese](http://japanese.engadget.com/2013/05/15/google-i-o-2013/)

[【速報】 Google I/O 2013キーノート講演まとめ #io13-ノート｜U-NOTE【ユーノート】-イベントまとめプラットフォーム](http://u-note.me/note/47484899)

[Google株、Google I/O初日に52週最高の915ドルで引ける。Apple再び下げる | TechCrunch Japan](http://jp.techcrunch.com/2013/05/16/20130515google-stock-price-closes-at-52-week-high-of-915-on-first-day-of-google-io-as-apple-takes-another-drop/)

#1.Android Studioインストール

もうWindows/Mac/Linux版が公開されています。ただしv0.1の*early access preview*なので、productionには使わない方が良いとのこと。

<!-- more -->

[Getting Started with Android Studio | Android Developers](http://developer.android.com/intl/ja/sdk/installing/studio.html)

(5/16追記) Macの場合、*/Applications ディレクトリにコピーしないとエラーが出る模様*。情報ありがとうございます！

インストールが完了して起動すると、IntelliJ IDEAっぽいスプラッシュ画面。

{% img /images/20130516/android_studio_splash.jpg 200 150 %}

生成されるプロジェクトはこんな感じ。

{% img /images/20130516/android_studio_project.jpg 400 300 %}

ビルドは[Gradle](http://www.gradle.org/)なんですねー。[Gradle Wrapper](http://d.hatena.ne.jp/bluepapa32/20110308/1299602195)つき。

	build.gradle
	settings.gradle
	gradlew
	gradlew.bat	

そしてIDEの設定ファイルは、IntelliJ IDEAと似たような構成みたい。

	.idea/
	<HOGEHOGE>.iml

#2.便利なPalette機能

	Androidアプリ開発者のために、複数のディスプレイサイズでのレイアウトをプレビューできるレイアウトエディタ<中略>が用意されている
	
とのことなので、早速チェックしてみることに。どうやらIntelliJ IDEAのAndroid UI Designerをベースにしつつ、機能追加/画面の修正をして*Palette*という名前に変更したようです。

{% img /images/20130516/android_studio_screens.jpg 400 300 %}

なんと、レイアウトファイルの**各解像度での見え方のプレビューを一覧表示できる！！**

**これは便利すぎる…**

#3.IntelliJ IDEAとの互換性

*  PluginはIntelliJ IDEAのものを一部(VCS系、Inspector系など)利用可能。**Scala pluginは使用不可**。
*  IDE設定ファイルは**IntelliJ IDEAとの互換性無し**。一応開けるけど、PaletteないしはAndroid UI Designer画面で、こんなエラーが発生する。IntelliJ IDEAベースで開発しつつ、Android Studioをレイアウトファイルのエディタとして使用するのは無理ぽ。
* **(5/18追記)**どうやらAndroidStudioは[IntelliJ IDEA 13 EarlyPreview版](http://blogs.jetbrains.com/idea/2013/05/intellij-idea-13-early-preview-is-out/)をベースにしている模様。IntelliJ v13のプロジェクトをAndroidStudioで開けるそうなので、AndroidStudioをレイアウトファイルエディタとして使うというのが現実味を帯びてきました(ぇ。	参考:[Android Studio試してみたよ - marsのメモ](http://d.hatena.ne.jp/masanobuimai/20130516/1368708416)

{% img /images/20130516/android_studio_error.jpg 400 300 %}

…Android Studioの*エラーの報告先がJetBrains(IntellIJ開発元)になってる*のは、すごく迷惑だと思うけどw

#4.その他雑感

*  Android Studio単体で使っててもちょこちょこエラーが発生するので、メインで使うのはまだ厳しいかもしれない。
*  というか、IntelliJ IDEA本体にAndroidの各解像度プレビュー機能を追加してくれるのが僕にとってはBEST (w
*	今後のアップデートに期待！