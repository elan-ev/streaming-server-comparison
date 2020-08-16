# Streaming Server Comparison

Compare [NGINX RTMP](https://www.nginx.com/products/nginx/modules/rtmp-media-streaming/) and [SRS](https://github.com/ossrs/srs).

## Preamble

It seems like both servers `nginx-rtmp.stream.uos.de` and `srs.stream.uos.de` have a broken IPv6 configuration.
The DNS entries have other IPv6 addresses configured than the VMs itself and they seem to be not reachable via IPv6.
So if there are any problems with accessing the VMs, fix the DNS entries, the VMs or, for testing, simply use the IPv4 address instead of the DNS name.

## Nginx RTMP

The configuration for the server `nginx-rtmp.stream.uos.de` is done with the playbook [nginx-rtmp.yml](./nginx-rtmp.yml) and is hopefully self-explanatory and sufficiently commented.

The NGINX RTMP plugin is configured via the nginx configuration and the full configuration for the server can be found in [nginx.conf.j2](templates/nginx-rtmp/nginx.conf.j2).
The enabled features and functionalities can best be seen directly in the configuration.

Scaling should be pretty straight, since functionalities from NGINX can be used for that.
For example there could be multiple server for receiving and transcoding streams, storing them on some kind of network storage and on the other end multiple NGINX server publishing those streams.

The number of different tutorials on how to use this plugin suggest that is is broadly in use.
For documentation I would point especially to the [plugin's wiki](https://github.com/arut/nginx-rtmp-module/wiki/Directives) and to a pretty extensive [blog entry from NGINX](https://www.nginx.com/blog/video-streaming-for-remote-learning-with-nginx/).

Most difficult part of the installation and configuration, is the fact that the plugin has to be compiled for the exact NGINX version.
This is solved by creating a rpm via [Fedora COPR](https://copr.fedorainfracloud.org/coprs/shaardie/nginx-rtmp/),
which build is a modified version of the NGINX packaging for Centos 8 and therefor match the exact version inclusive source patches.

Unfortunately it seems like the NGINX RTMP module does not support deliver different dash variants, but there are [forks](https://github.com/justcodingtv/nginx-rtmp-module) that support this.

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

