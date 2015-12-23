---
layout: post
title: "そこまで怖くないAndroid BLE"
date: 2015-12-24 00:08:16 +0900
comments: true
categories: android ble
---

これは[BLE Advent Calendar 2015](http://qiita.com/advent-calendar/2015/ble) 7日目の記事です。

AndroidのBluetooth Low Energy(BLE)について、どんなイメージをお持ちでしょうか？

安定してない？わかりにくい？

色々と怖いイメージをもたれがちと思います。確かに怖いところも色々とありますが、今ちょうどAndroid BLEをフルに使ったプロダクト[BONX -Wearable Walkie-Talkie-](http://bonx.co/ja/)を開発しているので、そこで溜まった知見を共有できればと思います。

<!--more-->

#BLEの基礎
BLEは、接続する２デバイス間の関係を明確に分かれています。CentralとPeripheralです。

| -役割- | -Central- | -Peripheral- |
| :-: | :-: | :-: |
| 動作 | Scan | Advertise |
| GATT | Client | Server |
| version | 4.3~ | 5.0~ |

#BLE機能の使用方法
まず、以下のpermissionとfeatureが必要になります。

```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```

Android 6.0から、以下のpermission(ACCESS_FINE_LOCATIONでも可)が必要になる上に、***更にユーザーに明示的に位置情報使用許可*** をもらう必要が有ります(iOSっぽいですね)。

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

使用許可は、以下のように求めましょう。許可を求めるDialogが表示され、一度許可をもらえば、その後はずっと（端末の設定で明示的にOFFにされない限り）有効です。

```java
public class MyActivity extends Activity{
   ....
   requestPermissions(new String[]{Manifest.permission.ACCESS_COURSE_LOCATION}, REQUEST_CODE)
   ...
   @Override
   public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults){
       if(requestCode == REQUEST_CODE)
       {
           ...
       }
   }
}
```

[Bluetooth Low Energy startScan on Android 6.0 does not find devices - Stack Overflow](http://stackoverflow.com/questions/33043582/bluetooth-low-energy-startscan-on-android-6-0-does-not-find-devices)

#BLE Central機能
Android 4.3から、BLE Central機能のサポートを開始しました。

##BLE Central機能で使うクラス,メソッド
Android 4.4以下(API Level 20以下)では[`BluetoothAdapter#startLeScan`](http://developer.android.com/intl/ja/reference/android/bluetooth/BluetoothAdapter.html)を使います。

Android 5.0以上(API Level 21以上)では先の旧APIはdeprecatedになっていますので、[`BluetoothAdapter#getBluetoothLeScanner`](http://developer.android.com/intl/ja/reference/android/bluetooth/BluetoothAdapter.html)からの[`BluetoothLeScanner#startScan`](http://developer.android.com/intl/ja/reference/android/bluetooth/le/BluetoothLeScanner.html)を使います。

5.0以上のほうが、[`ScanSettings`](http://developer.android.com/intl/ja/reference/android/bluetooth/le/ScanSettings.html)や、より細かな[`ScanFilter`](http://developer.android.com/intl/ja/reference/android/bluetooth/le/ScanFilter.html)を指定できたり、AdvertisePacketの中身を生byte列ではなく[`ScanResult`](http://developer.android.com/intl/ja/reference/android/bluetooth/le/ScanResult.html)で取得できたりと、柔軟性高く利用できます。

APIの使いやすさもそうですが、Android 4.3, Android 4.4ではBLE Central機能があまり安定していない（よく接続やRead/Writeに失敗するし、遅い）ので、注意してください。

##BLE Central実験用アプリ

iOS BLEテスト用アプリといえば、[Light Blue](https://itunes.apple.com/jp/app/lightblue-explorer-bluetooth/id557428110?mt=8)がデファクトだと思いますが、AndroidでCentral機能を[BLE Scanner: Read,Write,Notify](https://play.google.com/store/apps/details?id=com.macdom.ble.blescanner&hl=ja)です。

BLE CentralのWrite, Read, Notificationといった基礎機能を全て安定して使うことができますので、かつ安定しているのでテストによく使います。

##その他注意というかBad Know-How
* BLE接続機能の安定性は端末差が大きい

おおよその端末でBLE Scan自体は比較的安定しているのですが、BLE接続、具体的には接続にかかる時間及びRead/Writeの成功率には個体差が大きいです。

接続・切断処理を短時間の間に繰り返すアプリを作る場合は注意してください。

* BLEのOSキャッシュにより、長時間使用していると謎の挙動を示すことがある

具体的には、BLE接続に失敗する、ないしはRead /Writeに失敗する確率が上がる、などです。

そうなった場合は、まずBluetooth ON/OFFを試して、もしダメなら端末再起動すると直ります。

* BLE characteristicに書き込む値のENDIANに注意する。

BLEアプリを作る場合、大抵はクロスプラットフォーム（というかiOSで作っていて、そのAndroid版を作りたい場合）が多いと思います。

その場合は、ENDIANには注意してください。iOSはcharacteristicにはLITTLE ENDIANで読み書きする一方、AndroidはBIG ENDIANです。

* [`BluetoothDevice#getType`](http://developer.android.com/intl/ja/reference/android/bluetooth/BluetoothDevice.html)が`DEVICE_TYPE_DUAL`のデバイスへの接続が頻繁に失敗する

startLeScanで見つかったデバイスに対しても、Classic BTを接続を試みている？っぽいのが原因の模様。

[Issue 58942 - android - BluetoothDevice.connectGATT Will Not Connect to BluetoothDevice.DEVICE_TYPE_DUAL - Android Open Source Project - Issue Tracker - Google Project Hosting](https://code.google.com/p/android/issues/detail?id=58942)

[Dual-mode issue when connecting Android BLE - Google グループ](https://groups.google.com/forum/#!topic/btstack-dev/9zWiU5_V9MI)

#BLE Peripheral機能
Android 5.0から、OS上はBLE Peripheralのサポートを開始しました。

しかしながら、何故か現在使われている[Bluedroid](https://source.android.com/devices/bluetooth.html)というBluetoothプロトコル・スタックでは、Broadcom製のチップセットのHCIコマンド`multiple advertisement`に依存しており、端末が搭載しているチップセットによってはBLE Peripheral機能が使えません。

[bluetooth - Android 5.0でBLE advertising するための要件 - Qiita](http://qiita.com/eggman/items/6a13f5be7deb363c800d)

BLE Peripheral機能を使えるかどうかは、以下のように判定できますので、チェックしましょう。但し端末がAdvertiseをサポートしていても、BluetoothがOFFになっているとき(`BluetoothAdapter#isEnabled`がfalseのとき)は以下も`false`を返すことに注意してください。

```java
public boolean isCapableToAdvertise(BluetoothAdapter bAdapter){
return Build.VERSION.SDK_INT > 20 &&
     null != bAdapter.getBluetoothLeAdvertiser() &&
     bAdapter.isMultipleAdvertisementSupported() &&
     bAdapter.isOffloadedFilteringSupported() &&
     bAdapter.isOffloadedScanBatchingSupported();
}
```

[AndroidでBLEのAdvertise modeを使えるかどうかのメモ - Qiita](http://qiita.com/qjuliar/items/0dd825f25d9de40d041f#_reference-4eb57a712932c923b08b)

BLE Peripheral機能が使える端末のリストは、以下が一番まとまっていると思います。

[Android Beacon Library](http://altbeacon.github.io/android-beacon-library/beacon-transmitter-devices.html)

ここにない端末で言えば、Galaxy S6, Zenfone 2 Laser, Galazy S5 Active, Nexus 6PなどはAdvertiseに対応しています。

##BLE Peripheral機能で使うクラス、メソッド
GattServerの初期化は[`BluetoothManager#openGattServer`](http://developer.android.com/intl/ja/reference/android/bluetooth/BluetoothManager.html)から行います。

Advertiseの開始は[`BluetoothAdapter#getBluetoothLeAdvertiser`](http://developer.android.com/intl/ja/reference/android/bluetooth/BluetoothAdapter.html)からの[`BluetoothLeAdvertiser#startScan`](http://developer.android.com/intl/ja/reference/android/bluetooth/le/BluetoothLeAdvertiser.html)を使います。

##BLE Peripheral機能の諸注意

* [`AdvertiseData.Builder#setIncludeDevicename`](http://developer.android.com/intl/ja/reference/android/bluetooth/le/AdvertiseData.html)を`true`にすると、Advertiseが不安定になることがある。

不要な場合は、inludeするのをやめましょう。

#最後に

Android BLE機能についてざくっと紹介しましたが、いかがでしょうか。

色々バグがあったりクセがあったりして確かに扱いにくいとは思いますが、頑張れば対応できるレベルだと思います。

まだまだ開発しながらノウハウを貯めている段階ですので、もし間違いなどあればぜひ[@OE_uia](https://twitter.com/OE_uia)に教えてください。よろしくお願いします。
