---
title: "VirtualBoxの使い方"
date: 2018-09-14 00:00
tags:
- virtualbox
- linux
---

## 環境

```
Host OS : RedHat
Guest OS : Ubuntu
```

## コマンドライン

### ゲストOS起動

```shell
$ VBoxManage startvm --type headless <uuid|vmname>
```

### ステータス確認

```shell
$ VBoxManage showvminfo <uuid|vmname>
```

## Guest Additionsインストール

<https://www.virtualbox.org/manual/ch04.html>  
Device -> Guest Addons CD イメージの挿入

```shell
$ sudo apt install gcc make
$ mount /dev/cdrom /mnt
$ cd /mnt
$ sudo ./VBoxLinuxAdditions.run
$ sudo shutdown -r now
```

## 共有フォルダ

Guest Addonsを入れると使える。

```shell
$ mount -t vboxsf <share name> <mount poiht>
```

## 履歴

- 2018-12-15: scrapboxから転記加筆修正