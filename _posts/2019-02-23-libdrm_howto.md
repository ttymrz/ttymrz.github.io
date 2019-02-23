---
title: "libdrmの使い方"
date: 2019-02-23
tags:
- Linux
---

libdrmのドキュメントがあまりにも少ないので，使い方を調べるために [libdrm][libdrm] や [drm-howto][drm-howto] のソースコードを解析した。
Linuxのグラフィックを使いこなすにはkernelのdrm，グラフィックデバイスのドライバ，libdrmを理解する必要がある。

[libdrm]:https://cgit.freedesktop.org/mesa/drm/
[drm-howto]:https://github.com/ttymrz/drm-howto

## 参考

- [Direct Rendering Manager (DRM)](https://dri.freedesktop.org/wiki/DRM/)
- [Linux GPU Driver Developer’s Guide](https://dri.freedesktop.org/docs/drm/gpu/)
- [Direct Rendering Manager (Wikipedia)](https://en.wikipedia.org/wiki/Direct_Rendering_Manager)
- [LibDRM  01.org](https://01.org/linuxgraphics/community/libdrm)
- [The DRM/KMS subsystem from a newbie’s point of view(PDF)](https://events.static.linuxfound.org/sites/events/files/slides/brezillon-drm-kms.pdf)
- [NVIDIA DRIVE 5.0 Linux SDK API Reference Direct Rendering Manager](https://docs.nvidia.com/drive/nvvib_docs/NVIDIA%20DRIVE%20Linux%20SDK%20Development%20Guide/baggage/group__direct__rendering__manager.html)


## DRMの簡単な構成

```
framebuffer -> CRTC -> Encoder -> Connector -> Display
                 |
               planes
```

それぞれの勝手な解釈
- CRTCはCRT Controller
- ConnectorはHDMIやDisplayPortなど文字通りコネクタ
- Encoderはframe bufferをVideo信号に変換する仮想デバイス
- PlaneはFramebufferを含むメモリオブジェクト。デバイスが対応していれば複数持ってoverlayも可能。

libdrmはデバイス依存でnVidia用libdrmとかバリエーションがあるようだ？

## drm-howtoソースコード解析

[drm-howto][drm-howto] のソースコードを順番に解析する。
これでlibdrmの使い方は大体分かる。

### modeset_open

まずは `main` の中の `modeset_open` から。
`/dev/dri/card0` をopenしてfile descriptorを取得。
```c
fd = open(node, O_RDWR | O_CLOEXEC);
```

DUMB_BUFFERが使えるか確認。
```c
if (drmGetCap(fd, DRM_CAP_DUMB_BUFFER, &has_dumb) < 0 || !has_dumb)
```

DRM deviceの情報を取得。 CRTC, Connector, Encoderの情報が入っている。
```c
res = drmModeGetResources(fd);
```

### modeset_prepare

すべてのConnectorを調べる。
```c
for (i = 0; i < res->count_connectors; ++i)
```

Connectorの情報を取得。
```c
conn = drmModeGetConnector(fd, res->connectors[i]);
```

### modeset_setup_dev

Displayが接続されてるか確認。
```c
if (conn->connection != DRM_MODE_CONNECTED)
```

mode情報をコピーする。
```c
memcpy(&dev->mode, &conn->modes[0], sizeof(dev->mode));
```

`conn->modes` が `struct drmModeModeInfo` でDisplayで使える解像度情報が入っている。
つまりEDIDの情報。
`drmModeModeInfo` の `hdisplay`，`vdisplay` が解像度，`vrefresh` がフレームレート。

### modeset_prepare

現在Connectorに接続しているEncoderとCRTCが使用可能か調べる。
```c
/* first try the currently conected encoder+crtc */
if (conn->encoder_id)
	enc = drmModeGetEncoder(fd, conn->encoder_id);
else
	enc = NULL;
```

現在のEncoder+CRTCが使用不可だった場合は使えるEncoder+CRTCを探す。
```c
for (i = 0; i < conn->count_encoders; ++i)
```

### modeset_create_fb

使用可能なEncoder+CRTCがあったらframe bufferをつくる。

```c
/* create dumb buffer */
...
ret = drmIoctl(fd, DRM_IOCTL_MODE_CREATE_DUMB, &creq);
...
/* create framebuffer object for the dumb-buffer */
ret = drmModeAddFB(fd, dev->width, dev->height, 32, 32, dev->stride, dev->handle, &dev->fb);
...
/* prepare buffer for memory mapping */
memset(&mreq, 0, sizeof(mreq));
mreq.handle = dev->handle;
ret = drmIoctl(fd, DRM_IOCTL_MODE_MAP_DUMB, &mreq);
...
/* perform actual memory mapping */
dev->map = mmap(0, dev->size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, mreq.offset);
```

### 再びmain

選択したmodeと作ったframe bufferでCRTCを設定する。
```c
ret = drmModeSetCrtc(fd, iter->crtc, iter->fb, 0, 0, &iter->conn, 1, &iter->mode);
```

ここまでで描画の準備完了。

### modeset_draw 描画

frame bufferにデータを書き込む

#### modeset_draw (single bufferの場合)

```c
*(uint32_t*)&iter->map[off] = (r << 16) | (g << 8) | b;
```

#### modeset_draw (double bufferを使う場合)

frame bufferにデータを書き込み `drmModeSetCrtc` でframe bufferを切り替える。
```c
ret = drmModeSetCrtc(fd, iter->crtc, buf->fb, 0, 0, &iter->conn, 1, &iter->mode)
```

#### modeset_draw (vsyncで同期する場合)

`page_flip_handler` のevent handlerを使う。
```c
drmHandleEvent(fd, &ev);
```

`page_flip_event` が発生したらframebufferにデータを書き込み
`drmModePageFlip` で切り替える。
```c
ret = drmModePageFlip(fd, dev->crtc, buf->fb, DRM_MODE_PAGE_FLIP_EVENT, dev);
```

グラフィックデバイスのDMAの仕様によると思うが，
double bufferを使ってもV同期で切り替わっている気がする。
frame rateやframe切り替わりのタイミングをコントロールしたいときにvsync同期を使うのだろうか？

### modeset_cleanup お掃除

設定したCRTCを元の設定にもどす。
```c
/* restore saved CRTC configuration */
drmModeSetCrtc(fd,
               iter->saved_crtc->crtc_id,
               iter->saved_crtc->buffer_id,
               iter->saved_crtc->x,
               iter->saved_crtc->y,
               &iter->conn,
               1,
               &iter->saved_crtc->mode);
drmModeFreeCrtc(iter->saved_crtc);
```    

あとはframe bufferを開放したりデバイスをcloseしたり。

## drmModeAddFB2

`drmModeAddFB` の代わりに `drmModeAddFB2` をつかうとYUVやRGBといったpixel_formatの指定ができる。
pixel_format は fourcc codeで指定。forccの定義はlibdrmの `<drm/drm_fourcc.h>` に書かれている。
指定できるfourccはデバイス依存があるかもしれない。

`drmModeAddFB2`のprototypeは以下のようになっている。
```c
drm_public int drmModeAddFB2(int fd, uint32_t width, uint32_t height,
 		uint32_t pixel_format, const uint32_t bo_handles[4],
 		const uint32_t pitches[4], const uint32_t offsets[4],
 		uint32_t *buf_id, uint32_t flags)
```

`bo_handles`(buffer object)，`piches`，`offsets`は4つあるが，1つしか使わない場合は最初の1個だけ設定すればよいみたい。
2planeや3plane使うforcc指定があり，そのときに`bo_handles`を複数指定するようだ。

`include/drm/drm_mode.h` にbuffer objectを4つ指定できる理由が書いてあるので引用する。
```c
/*
 * In case of planar formats, this ioctl allows up to 4
 * buffer objects with offsets and pitches per plane.
 * The pitch and offset order is dictated by the fourcc,
 * e.g. NV12 (http://fourcc.org/yuv.php#NV12) is described as:
 *
 *   YUV 4:2:0 image with a plane of 8 bit Y samples
 *   followed by an interleaved U/V plane containing
 *   8 bit 2x2 subsampled colour difference samples.
 *
 * So it would consist of Y as offsets[0] and UV as
 * offsets[1].  Note that offsets[0] will generally
 * be 0 (but this is not required).
 *
 * To accommodate tiled, compressed, etc formats, a
 * modifier can be specified.  The default value of zero
 * indicates "native" format as specified by the fourcc.
 * Vendor specific modifier token.  Note that even though
 * it looks like we have a modifier per-plane, we in fact
 * do not. The modifier for each plane must be identical.
 * Thus all combinations of different data layouts for
 * multi plane formats must be enumerated as separate
 * modifiers.
 */
```


## 履歴

- 2019-02-23: 初稿
