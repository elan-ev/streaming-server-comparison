# Streaming Server Comparison

Compare [NGINX RTMP](https://www.nginx.com/products/nginx/modules/rtmp-media-streaming/) and [SRS](https://github.com/ossrs/srs).

## Nginx RTMP

### Create Stream

```bash
ffmpeg -re -f lavfi -i testsrc=r=25:s=1280x720,drawtext="fontfile=monofonto.ttf: fontsize=96: box=1: boxcolor=black@0.75: boxborderw=5: fontcolor=white: x=(w-text_w)/2: y=50: text='%{gmtime\:%H\\\\\:%M\\\\\:%S}'" -f lavfi -i sine=f=220:b=4 -vcodec libx264 -pix_fmt yuv420p -preset ultrafast -b:v 1500k -acodec aac -b:a 128k -f flv "rtmp://nginx-rtmp.stream.uos.de/live/mystream"
```

### View Stream

```bash
# MPEG-DASH
ffplay http://nginx-rtmp.stream.uos.de/live/dash/mystream.mpd
# HLS
ffplay http://nginx-rtmp.stream.uos.de/live/hls/mystream.m3u8
```

## Additional Sources

https://www.nginx.com/blog/video-streaming-for-remote-learning-with-nginx/

