---
title: Mac使用FFmpeg进行屏幕录制，并使用VLC本地播放
author: 雷子
date: 2025-11-20 13:00:00 +0800
categories: [技术, FFmpeg]
tags: [FFmpeg, 屏幕录制, VLC, Mac, nginx-full]
last_modified_at: 2025-11-20 13:00:00 +0800
---
这是一篇关于如何在`Mac`上使用`FFmpeg`进行屏幕录制，并使用`VLC`播放器进行本地播放的教程，默认读者已经安装好了`FFmpeg`和`VLC`播放器。
本文将介绍如何使用`FFmpeg`命令行工具进行屏幕录制，并通过`VLC`播放器进行实时播放，对于`FFmpeg`和`VLC`的安装和基础使用，读者可以参考相关文档。

## 1. Mac上捕获屏幕的FFmpeg输入源
在Mac上，`FFmpeg`可以使用`avfoundation`输入源来捕获屏幕内容。要查看可用的设备列表，可以运行以下命令：

```bash
ffmpeg -f avfoundation -list_devices true -i ""
```

你会看到类似如下的输出：

``` bash
AVFoundation video devices:
[0] Capture screen 0
[1] FaceTime HD Camera

AVFoundation audio devices:
[0] MacBook Pro Microphone
[1] Multi-Output Device
```

## 2. 直播Mac屏幕
假设你要推流到一个 RTMP 服务器（例：rtmp://localhost/live/stream）

### 2.1 不包含音频的屏幕录制命令：
```bash
ffmpeg \
  -f avfoundation -framerate 30 -i "0:none" \
  -c:v libx264 -preset veryfast -f flv rtmp://localhost/live/stream
```

### 2.2 包含音频的屏幕录制命令：
```bash
ffmpeg \
  -f avfoundation -framerate 30 -i "0:0" \
  -c:v libx264 -preset veryfast -maxrate 3000k -bufsize 6000k \
  -c:a aac -b:a 128k \
  -f flv rtmp://localhost/live/stream
```
解释：
- "0:0" → （屏幕设备 ID : 音频设备 ID）
- 0 = 主屏幕
- 0 = 麦克风
- -framerate 30 → 流畅度佳，CPU占用适中
- -f flv → RTMP 必须使用 FLV 封装
- libx264 → 网络直播最稳定

### 2.3 配置nginx-full
通常路径：
```bash
/opt/homebrew/etc/nginx/nginx.conf
```
在`nginx.conf`中添加RTMP模块配置：
```nginx
rtmp {
    http { ... }

    # 下面是要添加的内容
    server {
        listen 1935;

        application live {
            live on;
            record off;
        }
    }
}
```

### 2.4 启动nginx-full
```bash
brew services start nginx-full
```

### 2.5 验证RTMP是否启动成功
```bash
lsof -i :1935
```
如果看到 nginx 在监听该端口，说明启动成功
```bash  
nginx   12345   ...   LISTEN
```

### 2.6 推流FFmpeg
```bash
ffmpeg \
  -f avfoundation -framerate 30 -i "0:none" \
  -c:v libx264 -preset veryfast -f flv rtmp://localhost/live/stream
```

### 2.7 使用VLC播放RTMP流
打开VLC播放器，选择“打开网络串流”，输入以下URL：
```bash
rtmp://localhost/live/stream
```
点击“播放”，即可观看实时屏幕录制内容。

## 3.最后
在本文中，除了介绍如何使用`FFmpeg`在Mac上进行屏幕录制外，还涵盖了如何配置`nginx-full`以支持RTMP流的推送和播放。关于`nginx-full`的更多配置和使用，可以参考相关文档。