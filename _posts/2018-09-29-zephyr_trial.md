---
title: "Zephyrを試す"
date: 2018-09-29
tags:
- Zephyr
- RTOS
- STM32
---

## 開発環境

- Ubuntu 16.04
- zephyr-v1.13.0
- [STM32F3DISCOVERY]( https://www.st.com/ja/evaluation-tools/stm32f3discovery.html )
- Windows10 (書き込みだけ)

環境構築は基本的に
[Getting Started Guide]( https://docs.zephyrproject.org/latest/getting_started/getting_started.html )
を見ながら進める。

[CMake]( https://cmake.org/ )は最新版が必要。
aptではなく <https://cmake.org/> からダウンロードする。

pyhtonのパッケージはpipで最新版をインストールする。python-yamlも必要。

Zephyr SDKはなぜかビルドエラーになるので使えない。aptでgcc-arm-none-eabiをインストールする。

```shell
$ sudo apt-get install gcc-arm-none-eabi
```

## 環境設定

```shell
$ unset GNUARMEMB_TOOLCHAIN_PATH
$ export ZEPHYR_TOOLCHAIN_VARIANT=cross-compile
$ export CROSS_COMPILE=/usr/bin/arm-none-eabi-
```

## Hello Worldビルド

[ST STM32F3DISCOVERY]( https://docs.zephyrproject.org/latest/boards/arm/stm32f3_disco/doc/stm32f3_disco.html )
を見ながら進める。

```shell
$ cd $ZEPHYR_BASE/samples/hello_world
$ mkdir build && cd build
$ cmake -DBOARD=stm32f3_disco ..
```

KconfigのWarningが出たが無視しても大丈夫だった。

makeするとzephyr.elfやzephyr.binができあがる。
最後のほうにメモリサイズの情報が表示される。

```
Memory region         Used Size  Region Size  %age Used
FLASH:                  14868 B       256 KB      5.67%
SRAM:                    4856 B        40 KB     11.86%
IDT_LIST:                 120 B         2 KB      5.86%
```

### 書き込み
zephyr.binをST-LINKで書き込む。

### 実行
```
***** Booting Zephyr OS zephyr-v1.13.0 *****
Hello World! arm
```
UART1に表示される。

## オマケ
Lチカも作ってみたがGPIOがうまく動かない。device-tree書いたりデバイスドライバ設定したりひと手間必要なようだ。

## リンク

- <https://www.zephyrproject.org/>

## 履歴

- 2018-12-15: scrapboxから転記加筆修正
