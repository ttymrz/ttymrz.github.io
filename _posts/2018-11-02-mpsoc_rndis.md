---
title: "MPSoCでRNDISを動かす"
date: 2018-11-02
tags:
- MPSoC
- Xilinx
- Linux
---

## 概要
MPSoCのUSBをRNDIS deviceにしてUSB over Ethernetを実現する。
現時点では通信できるが動作が怪しい。

## 参考

- [Raspberry Pi Zero / Windows 10 automatic RNDIS driver install for composite gadgets]( https://gist.github.com/Gadgetoid/c52ee2e04f1cd1c0854c3e77360011e2#file-gadget-sh-L80 )
- <https://gist.github.com/Gadgetoid/c52ee2e04f1cd1c0854c3e77360011e2#file-gadget-sh-L80>

## Kernelのconfig

RNDISを有効にする
```
Device Drivers → USB support → USB Gadget Support  → RNDIS
```

## ConfigFSの設定
Vendor IDとProduct IDは拝借する。

```bash
#!/bin/sh

ID_VENDOR="0x0525"
ID_PRODUCT="0xa4a2"

cd /sys/kernel/config/usb_gadget

mkdir g1
cd g1

echo "0x0320" > bcdUSB
echo "0xEF" > bDeviceClass
echo "0x02" > bDeviceSubClass
echo "0x01" > bDeviceProtocol
echo $ID_VENDOR > idVendor
echo $ID_PRODUCT > idProduct

# Windows extensions to force config

echo "1" > os_desc/use
echo "0xcd" > os_desc/b_vendor_code
echo "MSFT100" > os_desc/qw_sign

mkdir strings/0x409
echo "9112473" > strings/0x409/serialnumber
echo "Netchip Technology, Inc." >strings/0x409/manufacturer
echo "Linux-USB Ethernet/RNDIS Gadget" >strings/0x409/product

# Config #1 for OSX / Linux

mkdir configs/c.1

mkdir functions/rndis.usb0
echo E0 > functions/rndis.usb0/class
echo 01 > functions/rndis.usb0/subclass
echo 03 > functions/rndis.usb0/protocol

# Set up the rndis device only first

ln -s functions/rndis.usb0 configs/c.1

# Tell Windows to use config #2

ln -s configs/c.1 os_desc

# Show Windows the RNDIS device

echo "fe200000.dwc3" > UDC
```

## 接続
WindowsもしくはLinuxとUSBで接続したらネットワークを有効にする。
```shell
$ ip addr add 192.168.10.10/24 dev usb0
$ ip link set usb0 up
```
Windowsのドライバは自動的にインストールされる。

## 性能

MPSoC <-> Linux
```shell
$ iperf -c 192.168.10.10
------------------------------------------------------------
Client connecting to 192.168.10.10, TCP port 5001
TCP window size: 85.0 KByte (default)
------------------------------------------------------------
  3 local 192.168.10.11 port 53598 connected with 192.168.10.10 port 5001
 ID Interval       Transfer     Bandwidth
  3  0.0-10.0 sec   362 MBytes   303 Mbits/sec
```

MPSoC <-> Windows
```shell
$ iperf -c 192.168.10.10
------------------------------------------------------------
Client connecting to 192.168.10.10, TCP port 5001
TCP window size: 63.0 KByte (default)
------------------------------------------------------------
  3 local 192.168.10.11 port 59641 connected with 192.168.10.10 port 5001
 ID Interval       Transfer     Bandwidth
  3  0.0-10.0 sec   653 MBytes   547 Mbits/sec
```

## 課題

- 設定が謎。
- MPSoCからWindowsが見えない。WindowsからはMPSoCは見えて通信できる。
- MPSoCとLinuxならどちらからも通信できる。
- USB3.0なのに通信スピードが遅い。接続先（PCの機種，OS）によってかなり変わる。

## 履歴

- 2018-12-15: scrapboxから転記加筆修正