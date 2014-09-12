---
layout: post
title: "ScalaでAndroidアプリを作るには（pfn/android-sdk-plugin）"
date: 2014-03-13 06:06
comments: true
categories: scala android intellij sbt
---
表題の件ですが、今は**[pfn/android-sdk-plugin](https://github.com/pfn/android-sdk-plugin)を使うのがオススメです**。

なぜか。

<!-- more -->

2013年6月に[ScalaでAndroidアプリを作るには](/blog/2013/06/08/scala-android-intellij/)という記事を書いたのですが、その後Android SDKのディレクトリ構造が(r22〜？)変わったものの、記事中で使用していた[jberkel/android-plugin](https://github.com/jberkel/android-plugin)がその変更に追従しておらず、*Could not find tool "aapt"*エラーが出てビルドできない問題が発生しています。

参考のSO記事はこちら。mavenのpluginの話ですが同じ問題。

[maven - Could not find tool aapt. Please provide proper Android SDK directory path as configuration parameter - Stack Overflow](http://stackoverflow.com/questions/16927306/could-not-find-tool-aapt-please-provide-proper-android-sdk-directory-path-as-co)

#pfn/android-sdk-pluginの、[jberkel/android-plugin](https://github.com/jberkel/android-plugin)との主な違い

* Proguardしたclassファイルをキャッシュしてくれるので、**デバッグ時のビルド時間が半分くらい**になる。
* ディレクトリ構造がsbtではなく、Android SDKデフォルト(android create project ...)。
* local.propeties,project.properties,proguard-project.txtなどAndroidSDKの各種configファイルも使用可能(なのでbuild.sbtが膨れ上がらずに済む)。
* Android NDKサポートがない
* proguardのデフォルトの設定が違うので、書き直しが必要（これが面倒…）

また思い出したら上に書き足します。

#pfn/android-sdk-pluginの問題

* sampleコードが古い
* configファイルなどを更新した場合も、reloadが必要
* ScalaTestなどScalaのテストフレームワークと相性悪い？（回避方法模索中）

#AndroidアプリをScalaで書くときの問題

あと、AndroidアプリをScalaで書くと、以下の問題にぶつかりやすくなります（DalvikVM自身の問題も多分に含まれるのですが）。

* DalvikVMのmethod数上限65,535個に引っかかる
* DalvikVMのLinearAllocの上限を超えてinstall_failed_dexoptエラー
* Scalaの変なバグ(e.g.[こんなの](/blog/2013/03/22/scala-2-dot-10-bytecode/))を踏みやすくなる...けど、そのぶんScalaに貢献できる（Scalazみたいに！）
* あんまりimmutable collectionを多用するとGCが頻繁走ってカクカクする。mutable collectionを多用しつつandroid:largeHeap="true"推奨？

ちなみにLinearAlloc問題が起きるのは基本的にAndroid2.3以下だし、DalvikVMに代わるARTがAndroid4.4から試験導入されているので、時間とともに改善するという説もあります。