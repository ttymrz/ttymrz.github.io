---
title: "M5StickC 2JCIE-BU01 GCP"
date: 2021-03-06
last_modified_at: 2021-03-06
tags:
- IoT
- M5Stack
- GCP
---

M5StickCで2JCIE-BU01のデータを収集してGCP IoT Coreに送信するアプリケーションを作った記録。
のつもりだったけどWiFiが不安定すぎて使い物にならなかった記録。

## 材料

- [M5StickC](https://docs.m5stack.com/#/en/core/m5stickc)
- OMRON [2JCIE-BU01](https://www.omron.co.jp/ecb/product-detail?partNumber=2JCIE-BU)
- GCPアカウント

## 構成

{% include figure image_path="/assets/images/20210306_m5stickc_2jciebu01_gcp.png"
alt="M5StackC 2JCIE-BU01 GCP IoT Core Block Diagram"
caption="M5StackC 2JCIE-BU01 GCP IoT Core Block Diagram" %}

## ライブラリ

- [M5StickC](https://github.com/m5stack/M5StickC) : 0.2.0
- [arduino-esp32](https://github.com/espressif/arduino-esp32) : 1.0.4
- [arduino-mqtt](https://github.com/256dpi/arduino-mqtt) : 2.5.0
- [Google Cloud IoT Core JWT](https://github.com/GoogleCloudPlatform/google-cloud-iot-arduino) : 1.1.11

## BLE

OMRON 2JCIE-BU01はAdvertising Packetにセンサデータが含まれているのでサーバに接続する必要がない。


センサデータはAdvertising PacketのManufacturer Specific Dataに含まれている。
Manufacturer Specific DataはBLEAdvertisedDevice Classの`getPayload()`か`getManufacturerData()`で取り出すことができる。
`getManufacturerData()`は何故かString型を返すので今回は`getPayload()`を使用した。

```c++
class MyAdvertisedDeviceCallbacks : public BLEAdvertisedDeviceCallbacks
{
	void onResult(BLEAdvertisedDevice advertisedDevice)
	{
		String deviceAddress = advertisedDevice.getAddress().toString().c_str();
		if (deviceAddress.equalsIgnoreCase(omronSensorAddress))
		{
			uint8_t *payload = advertisedDevice.getPayload();
			size_t paylen = advertisedDevice.getPayloadLength();
		}
	}
};
```

## Google Cloud IoT JWT

<https://github.com/GoogleCloudPlatform/google-cloud-iot-arduino/tree/master/examples/Esp32-lwmqtt>
を参考にただ使うだけ。

[arduino-esp32](https://github.com/espressif/arduino-esp32)
が1.0.5だと動かなかったので1.0.4を使った。

## LCD

2JCIE-BU01のセンサデータを表示することにしたが常に見るわけではないのでLCDはButtonを押したときに表示して一定時間でOFFするようにした。
消費電力を落としつつすぐ復帰できるようにするためにはLCD ControllerのST7735SVをSleep ModeにしてバックライトをOFFにすればよい。これでST7735SVのメモリを保持したままLCDの電源をOFFにできる。

### Display OFF

```c++
// sleep display and turn off backlight
M5.Axp.SetLDO2(false);
M5.Lcd.writecommand(ST7735_SLPIN);
```

### Display ON

Sleep Modeを解除した後に120msの待ち時間が必要。

```c++
// wake up display and turn on back light
M5.Lcd.writecommand(ST7735_SLPOUT);
// it is necessary to wait 120msec before sending next command
// because of the stabilization timing for the supply voltages and clock circuits.
delay(120);
M5.Axp.SetLDO2(true);
```

AXP192のAPIに`M5.Axp.SetSleep()`というAPIがあるがこれはLCDとST7735SVの電源が両方落ちてしまうので復帰させるときに再初期化が必要。ちょっと試してみたところうまく復帰できなかった。

## Button

約100ms周期でM5.update()を呼び出すと安定してイベントが取得できた。これより長いと反応が鈍くなるようだ。


## メモリ使用量

WiFiとBLEが大きいのでPartition SchemeはNO OTA(Large APP)を選択しないと入らない。
ビルド結果は以下のようになった。

```
最大2097152バイトのフラッシュメモリのうち、スケッチが1563078バイト（74%）を使っています。
最大327680バイトのRAMのうち、グローバル変数が62612バイト（19%）を使っていて、ローカル変数で265068バイト使うことができます。
```

## WiFIの安定性

WiFiとBLEを同時につかうとWiFiが不安定になるようだ。
WiFiだけ使う場合はそこそこ安定していた。
BLEはWiFiが動いていても安定していた。

## ソースコード

<https://github.com/ttymrz/m5stickc_2jciebu01_gcp>

## リンク

- <https://docs.m5stack.com/#/en/core/m5stickc>
- <https://github.com/m5stack/M5StickC>
- <https://github.com/m5stack/M5-Schematic>
- M5StickC非公式日本語リファレンス <https://lang-ship.com/reference/unofficial/M5StickC/>
- <https://github.com/nkolban/ESP32_BLE_Arduino>
- BLE doc <https://github.com/nkolban/esp32-snippets/tree/master/Documentation>
