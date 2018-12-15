---
title: "NuttXを試す"
date: 2018-09-19
tags:
- NuttX
- STM32
- RTOS
---

## 開発環境

- Ubuntu 16.04
- [STM32F3DISCOVERY](https://www.st.com/ja/evaluation-tools/stm32f3discovery.html )
- Windows10 (書き込みだけ)

## kconfig-frontendsのインストール

```shell
$ apt install automake gperf flex bison libncurses-dev

$ git clone https://bitbucket.org/nuttx/tools
$ cd tools kconfig-frontends
$ ./configure
$ make
$ make install
```

## tool chainのインストール

```shell
$ apt install gcc-arm-none-eabi
```

## NuttX
[NuttX wiki](http://nuttx.org/doku.php?id=wiki) を参考に進めればよい。  
[Getting Started with NuttX -- STM32F4 Discovery]( http://nuttx.org/doku.php?id=wiki:getting-started:stm32f4discovery_unix )

```shell
$ git clone https://bitbucket.org/nuttx/nuttx.git
$ git clone https://bitbucket.org/nuttx/apps.git

$ cd nuttx
$ git checkout nuttx-7.26

$ cd apps
$ git checkout nuttx-7.26
```

### Configuration

```shell
$ cd nuttx
$ ./tools/configure.sh -l stm32f3discovery/usbnsh
```

### Build
```shell
$ make
```

nuttx.hexとnuttx.binができるので[ST-LINK utility]で書き込む。  
nuttx.binは57kBだった。

[ST-LINK utility]: <https://www.st.com/ja/development-tools/stsw-link004.html>


## STM3F3DISCOVERY

ST-LINKのファームウェアアップグレードが必要だった。  
[STSW-LINK007]( https://www.st.com/content/st_com/ja/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-programmers/stsw-link007.html )
ここからダウンロードする。

### NuttX Boot

BootするとUSB COM portでアクセスできる。起動まで数秒かかった。USBの認識に時間がかかるのか？

```shell
NuttShell (NSH)
nsh> help
help usage:  help -v <cmd>

        [           cp          exit        mb          rm          time
        ?           cmp         false       mkdir       rmdir       true
        basename    dirname     help        mh          set         uname
        break       dd          hexdump     mv          sh          unset
        cat         echo        kill        mw          sleep       usleep
        cd          exec        ls          pwd         test        xd

Builtin Apps:
nsh>
```

USBのIDは以下を借用していた。

```
VID: 0525 (Netchip Technology, Inc.)
PID: A4A7 (Linux-USB Serial Gadget (CDC ACM mode))
```

デバイスドライバは自動的にインストールされた。

### Factory Image
工場出荷時のファームウェアは
[STSW-STM32118]( https://www.st.com/content/st_com/ja/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-standard-peripheral-library-expansion/stsw-stm32118.html )
に入っている。

## リンク

- <http://nuttx.org/>

## 履歴

- 2018-12-15: scrapboxから転記加筆修正
