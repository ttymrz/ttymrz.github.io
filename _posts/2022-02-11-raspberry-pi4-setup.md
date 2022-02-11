---
title: "Raspberry Pi 4 Setup"
date: 2022-02-11
last_modified_at: 2022-02-11
tags:
- Raspberry Pi
- Linux
---

Raspberry Pi 4の性能を引き出すために公式ドキュメントに書かれていない設定や調査に時間が掛かった部分があったのでまとめておく。

## 公式ドキュメントとgithub

- [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)
- [GitHub Raspberry Pi](https://github.com/raspberrypi)

## Bootloader アップデート

Raspberry Piを購入したらまず最初にbootloaderをアップデートする。

### Version確認

```console
$ vcgencmd bootloader_version
$ sudo rpi-eeprom-update
```

### アップデート

```console
$ sudo rpi-eeprom-update -a
$ sudo reboot
```

リリースノートは以下のリンクにある。
- [https://github.com/raspberrypi/rpi-eeprom/releases](https://github.com/raspberrypi/rpi-eeprom/releases)

## vcgencmd

設定では無いがRaspberry Piの各種ステータスを確認するコマンド。

```console
$ sudo vcgencmd commands
```
でコマンド一覧がわかる。

ソースコードはこちら。
- [https://github.com/raspberrypi/userland/tree/master/host_applications/linux/apps/gencmd](https://github.com/raspberrypi/userland/tree/master/host_applications/linux/apps/gencmd)

## Watchdog

Watchdogを有効にするには`config.txt`に以下の行を追加する。

```
dtparam=watchdog=on
```

Watchdogはheartbeatとnowayoutの2つのパラメータを持っている。
heartbeatのdefault値は15秒で最大値も15秒。
この値はdevice driver
[`bcm2835_wdt.c`](https://github.com/raspberrypi/linux/blob/rpi-5.10.y/drivers/watchdog/bcm2835_wdt.c)
を解析して判明した。
nowayoutのdefault値は0。nowayoutに1を設定するとWatchdogを停止できなくなる。

パラメータは通常default値で使えばよいと思うが変更したい場合は`cmdline.txt`に以下の文字列を追加する。

```
bcm2835_wdt.heartbeat=10 bcm2835_wdt.nowayout=1
```

heartbeatの機能はsystemdが持っているので`/etc/systemd/system.conf`に以下の設定を追加する。
`RuntimeWatchdogSec`にはbcm2835_wdt.heartbeatより小さい値を設定すればよい。

```
RuntimeWatchdogSec=10
```

## WiFiとBluetooth無効化

WiFiとBluetoothを使わない場合は`config.txt`に以下を追加すれば無効にできる。

```
dtoverlay=disable-bt
dtoverlay=disable-wifi
```

## GPUとメモリ最適化

Raspberry Piはheadlessで使う場合でもGPUをoffにできないようだ。
そこでGPUなどのハードウェアアクセラレータに割り当てるメモリを最小にする。
具体的には`config.txt`に以下の行を追加する。

```
dtoverlay=cma,cma-64
```

`cma-64`というパラメータがハードウェアアクセラレータに割り当てるメモリサイズ。もっと小さくできるかもしれないが0にはできない。
この値はCodecやカメラモジュールにも影響する。
詳細はkernelソースツリーに含まれている
[overlays/README](https://github.com/raspberrypi/linux/blob/rpi-5.10.y/arch/arm/boot/dts/overlays/README)
参照。

kernelが確保したCMAサイズを確認するには以下のコマンドを実行する。
```console
$ cat /proc/meminfo | grep CmaTotal
```

## Overclock

⚠**製品保証と動作温度に注意**⚠

Overclockするには`config.txt`に以下を記述する。
周波数はMHzの単位で指定する。下記設定の場合は2000MHzとなる。
最新のfirmwareでは`over_voltage`を設定してはいけない。設定するとDVFSが無効になる。

```
arm_freq=2000
```

常に `arm_freq` に設定した周波数で動かしたい場合は `force_turbo` を設定する。

```
force_turbo=1
```

実際の動作周波数を確認するには以下のコマンドを実行する。

```console
$ sudo vcgencmd measure_clock arm
```

Raspberry Pi 4BではCPUの周波数以外にGPUなどの周波数が設定できる。
