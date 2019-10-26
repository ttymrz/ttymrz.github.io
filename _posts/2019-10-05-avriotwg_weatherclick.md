---
title: "AVR-IoT WG + Weather Click"
date: 2019-10-05
last_modified_at: 2019-10-25
tags:
- IoT
- AVR
- GCP
---

AVR-IoT WGとWeather Clickボードの開発環境構築と動作確認の記録。

## 購入したもの

- [AVR-IoT WG][AVR-IoT WG]
- [Weather Click][Weather Click]
- 8pin ピンソケット

[AVR-IoT WG]:https://www.microchip.com/DevelopmentTools/ProductDetails/AC164160
[Weather Click]:https://www.mikroe.com/weather-click

## 基板の状態

購入直後の基板は次のようになっていた。

{% include figure image_path="/assets/images/191015_avriot_org.jpg" alt="Original AVR IoT WG and Weather Click" caption="Original AVR IoT WG and Weather Click" %}

購入前にいくつか不明だったところは以下のようになっていた。

- AVR-IoT WGのLiPo Connectorは実装されていた
- Weather Clickのピンヘッダは実装されていた
- 写真に写っているもの以外の付属品は無い

## AVR IoT WGの動作確認

AVR IoTはドキュメントの通りにWi-Fiの設定をすればすぐに動く。
Wi-Fiの設定をしたら <https://www.avr-iot.com/> でLight SensorとTemperature Sensorの値が確認できる。

温度は基板の温度なので，だいたい気温より7,8度高く出るようだ。
Light Sensorはフリッカーを拾ってしまうので光源によっては使えないか後処理が必要になる。


### Pin Socket実装

一通り動作確認できたら[mikro BUS][mikro BUS]に拡張ボードを装着できるようにPin Socketをはんだ付けする。
Pin Socketは付属していないので別途購入する。

はんだ付けした基板は次のようになる。

{% include figure image_path="/assets/images/191015_avriot_wc_ps.jpg" alt="AVR IoT WG Pin Socket" caption="AVR IoT WG Pin Socket" %}

[mikro BUS]:https://www.mikroe.com/mikrobus

## Weather Clickの動作確認

Weather Clickを動かすにはファームウェアを準備しなければならない。
なので開発環境を構築する。

AVRの開発環境は[Atmel Studio 7][Atmel Studio 7]と
[MPLAB X IDE][MPLAB X IDE]で2種類ある。
どちらでも動作するのは確認したが
ここでは[Atmel Studio 7][Atmel Studio 7]の方法を記載する。

[Atmel Studio 7]:https://www.microchip.com/mplab/avr-support/atmel-studio-7
[MPLAB X IDE]:https://www.microchip.com/mplab/mplab-x-ide

### Atmel Studio 7のインストール

[Atmel Studio 7][Atmel Studio 7]のWebサイトから
web installerかoffline installerをダウンロードしてインストールすれば良い。
ウィルスチェックソフトとの相性があるのかインストールの途中でエラーになってしまうPCが2台あった。

AVR IoT WGだけ使えればよいのでArchitectureはAVR 8-bit MCUだけ選べばよい。

### ファームウェアの入手

[Atmel START][Atmel START]にExample projectがあるのでそれを使う。

[Atmel START][Atmel START]にアクセスして`BROWSE EXAMPLES`をクリックする。
`IoT`で絞りこむとWeather Clickに対応したprojectが見つかるので
そのままダウンロードすればよい。

ファイルはatzipという拡張子だが，ただのzipのようだ。

[Atmel START]:https://start.atmel.com/

### ファームウェアのビルド

[Atmel START][Atmel START]からダウンロードしたatzipをimportしてそのままビルドすればよい。

ビルドするとhexファイルができあがる。
プログラムサイズは以下のようになった。

```
  text     data     bss     dec     hex filename
 46788      354    2135   49277    c07d AVRIoTWGSensorNodewithWeatherClick1.elf
```

```
Program Memory Usage  : 43858 bytes   89.2 % Full
Data Memory Usage     : 2489 bytes    40.5 % Full
```

### www.avr-iot.com で確認

hexファイルをAVR-IoT WGに書き込んで再起動すると
<https://www.avr-iot.com/>
でWeather Clickのデータが確認できる。

Example projectそのままだとWeather Clickの温度が出てこない。
気圧の単位はkPaだった。

## カスタマイズ

### Weather Clickの温度追加

デバイスドライバはすでに実装してあるので
`Weather_getTemperatureDegC` 
を呼び出してjsonに付け加えるだけ。

### 送信間隔の変更

初期値は1秒と短いのでもっと長くする。

`IoT_Sensor_Node_config.h` に
`CFG_SEND_INTERVAL` というdefineがあるので適当に変更する。
あまり長くすると <https://www.avr-iot.com/> で表示がtimeoutして見れなくなってしまう。

ここまで変更したソースコードを
<https://github.com/ttymrz/AVRIoTWGSensorNodewithWeatherClick>
にアップしておく。

<https://www.avr-iot.com/> では簡単な動作確認しかできないので
ここから先はGCPに接続して進める。


## リンク

- <https://www.microchip.com/DevelopmentTools/ProductDetails/AC164160>
- <https://www.avr-iot.com/>
- <https://start.atmel.com/>
- <https://www.mikroe.com/weather-click>


## 履歴

- 2019-10-25: 初稿
