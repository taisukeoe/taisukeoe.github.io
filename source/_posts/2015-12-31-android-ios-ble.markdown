---
layout: post
title: "AndroidとiOSのBLE仕様で注意すべき点"
date: 2015-12-31 23:50:56 +0900
comments: true
categories: android ble
---
これは[BLE Advent Calendar 2015](http://qiita.com/advent-calendar/2015/ble) 7日目の記事です。

先日はAndroidのBLEにつきまして、記事を書きました。

[そこまで怖くないAndroid BLE - OE_uia Tech Blog](/blog/2015/12/24/android-ble/)

今回は、AndroidとiOSがBLE連携する際に気をつけるべき点を挙げたいと思います。

##Advertising Packet内のService UUIDは過信しない

<!--more-->

BLE Advertiseには、Advertising Packetと呼ばれる上限31byteのデータをAdvertiseに含めることができます。

このAdvertising Packetは規格化されており、Service UUIDやlocal name、manufacturer dataなどを含めるようにデザインされています。重要な点として、BLE CentralからBLE PeripheralのAdvertising PacketはBLE接続せずに取得することができるので、BLE Scanのフィルタリングに使えます。

例えば、特定のService UUIDをAdvertising Packetに含んでいるBLE PeripheralのみBLE Scanに引っかかるようにすることが出来ます。このUUID指定のBLE Scanは、高速に意中のBLE Advertiseを発見するために非常に効果的です。

...しかし、***AndroidとiOSが混在する場合で、なおかつiOSアプリがバックグラウンド状態でもAdvertiseする場合は注意*** が必要です。

なぜなら、iOSデバイスからのBLE Advertiseは、アプリがバックグラウンド状態ではAdvertising Packetからlocal nameとUUIDが抜け落ちるという、独特の癖が有ります。(バグかと思ったら、ドキュメント化もされており公式な仕様のようです。)

しかし、他のiOSデバイスからは、この癖については特に意識することなくBLE Scanで見つけることができたりするのです。どうやら、iOS独自の `overflow` 領域にAdvertising Packetの情報を保存しており、他のiOSアプリからはその `overflow` 領域も含めてBLE Scan時に参照することができ、結果として今Service UUIDを保存しているのがAdvertising Packetなのか `overflow` 領域かを意識せずにUUID指定のスキャンができます。

[Advertising in background, getting UUID | Apple Developer Forums](https://forums.developer.apple.com/thread/11705)

しかし、AndroidはこのiOSの `overflow` 領域を参照することができません。つまり、iOSがBLE Peripheralで、かつAdvertiseしているアプリがバックグラウンドにいってしまった場合、***CentralのAndroidからUUID指定のBLE ScanでiOSを見つけることができなくなります*** 。

すなわち、アプリがBLE Advertiseしたままバックグラウンドに行くことが前提の場合、**Central側のAndroidはUUID指定せずにBLE Scan** をしなければいけません。その一方で、iOSは気にせずUUID指定してBLE Scanできます。

## BLE characteristicのエンディアンに注意

iOSがBLE Characteristicにデータを書き込む場合、NSDataをコンテナとして使用してバイト列を書き込むと思うのですが、その場合バイト列のorderはリトル・エンディアンです。これはLightBlueなどを使って確認できます。

その一方、Androidのbyte orderは(少なくとも [`BluetoothGattCharacteristic#getValue`](http://developer.android.com/intl/ja/reference/android/bluetooth/BluetoothGattCharacteristic.html) から取得できるバイト列は)ビッグ・エンディアンなので、Characteristicを介してバイト列の受け渡しをする場合は、どこかでbyte orderを反転してあげる必要が有ります。

これについては決めごとの問題でしか無いので、BLE Characteristic上でバイト列を保持するのがビッグ・エンディアンなのか、リトル・エンディアンなのか、仕様としてまず最初に決めておくと良いと思います。

エンディアンについてはこちら。

[エンディアン - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%B3%E3%83%87%E3%82%A3%E3%82%A2%E3%83%B3)

この記事について、間違いなどあれば[@OE_uia](https://twitter.com/OE_uia)に教えてください。よろしくお願いします。
