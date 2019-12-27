# Nodejs实现直播功能

## 实现流程

Windows下用**node-media-server + ffmpeg + VlC/flv.js**搭建直播环境 实现推流、拉流 。

## [搭建流媒体服务器](https://github.com/illuspas/Node-Media-Server/blob/master/README_CN.md)

```bash
npm install node-media-server
```

```js
const NodeMediaServer = require('node-media-server');

const config = {
  rtmp: {
    port: 1935,
    chunk_size: 60000,
    gop_cache: true,
    ping: 30,
    ping_timeout: 60
  },
  http: {
    port: 8000,
    mediaroot: './media',
    allow_origin: '*'
  },
  // 转 HLS/DASH 直播流
  trans: {
    ffmpeg: 'D:/Program Files (x86)/ffmpeg/bin/ffmpeg.exe',
    tasks: [
      {
        app: 'live',
        hls: true,
        hlsFlags: '[hls_time=2:hls_list_size=3:hls_flags=delete_segments]',
        dash: true,
        dashFlags: '[f=dash:window_size=3:extra_window_size=5]'
      }
    ]
  }
};

var nms = new NodeMediaServer(config)

nms.run();
```

## 推流

推流工具：OBS（可以用于 win、linux、Mac）、RTMP 推流摄像机、FFmpeg。

**直播流推送到流媒体服务器之后，由流媒体服务器转发到连接的客户端。**

```bash
# 下载之后将ffmpeg解压到指定目录，并且配置环境变量

# BisonCam, NB Pro：照相机设备名称

# 麦克风 (Realtek High Definition Audio)： 麦克风名称

# rtmp://172.19.9.147:1935/live/home 流媒体服务器地址（home：STREAM_NAME）
```

![mark](http://gongyz.oss-cn-shenzhen.aliyuncs.com/blog/20191227/153217720.png)

```bash
# 使用 FFmpeg 推流
ffmpeg -f dshow -i video="BisonCam, NB Pro":audio="麦克风 (2-Realtek High Definition Audio)" -vcodec libx264 -acodec copy -preset:v ultrafast -tune:v zerolatency -f flv "rtmp://192.168.242.60:1935/live/home"
```

## 拉流

### VLC拉流播放

打开下载的 VLC，**点击媒体->打开网络串流->输入地址rtmp://172.19.9.147:1935/live/home->点击播放**

![mark](http://gongyz.oss-cn-shenzhen.aliyuncs.com/blog/20191227/153433166.png)

### 网页拉流播放

```js
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title></title>
  </head>
  <body>
    <script src="https://cdn.bootcss.com/flv.js/1.5.0/flv.js"></script>
    <video id="videoElement" style="width: 80%;" controls="controls"></video>
    <script>
      // if (flvjs.isSupported()) {
      //   var videoElement = document.getElementById('videoElement')
      //   var flvPlayer = flvjs.createPlayer({
      //     type: 'flv',
      //     url: 'http://192.168.242.60:8000/live/home.flv'
      //   })
      //   flvPlayer.attachMediaElement(videoElement)
      //   flvPlayer.load()
      //   flvPlayer.play()
      // }

      if (flvjs.isSupported()) {
        var videoElement = document.getElementById('videoElement');
        var flvPlayer = flvjs.createPlayer({
            type: 'flv',
            url: 'ws://192.168.242.60:8000/live/home.flv'
        });
        flvPlayer.attachMediaElement(videoElement);
        flvPlayer.load();
        flvPlayer.play();
      }
    </script>
  </body>
</html>

```

### ffplay拉流播放

```bash
# ffplay 是FFmpeg自带的一个小工具

# rtmp 流格式
ffplay rtmp://192.168.242.60:1935/live/home

# http-flv 流格式
ffplay http://192.168.242.60:8000/live/home.flv
```