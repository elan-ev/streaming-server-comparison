# Streaming Server Comparison

Compare [NGINX RTMP](https://www.nginx.com/products/nginx/modules/rtmp-media-streaming/) and [SRS](https://github.com/ossrs/srs).

## Preamble

It seems like both servers `nginx-rtmp.stream.uos.de` and `srs.stream.uos.de` have a broken IPv6 configuration.
The DNS entries have other IPv6 addresses configured than the VMs itself and they seem to be not reachable via IPv6.
So if there are any problems with accessing the VMs, fix the DNS entries, the VMs or, for testing, simply use the IPv4 address instead of the DNS name.

The setups are not ready for production. So, e.g., there is no TLS, and pushing RTMP streams to the servers is possibly from everywhere.
So the services `nginx` and `srs` respectively are stopped manually after the installation.
Also some manual steps were done on the VMs to ease the development, like installing administrative tools like `net-tools` or `policycoreutils-python-utils`.

## Nginx RTMP

The configuration for the server `nginx-rtmp.stream.uos.de` is done with the playbook [nginx-rtmp.yml](./nginx-rtmp.yml) and is hopefully self-explanatory and sufficiently commented.

The NGINX RTMP plugin is configured via the nginx configuration and the full configuration for the server can be found in [nginx.conf.j2](templates/nginx-rtmp/nginx.conf.j2).
The enabled features and functionalities can best be seen directly in the configuration.

Scaling should be pretty straight, since functionalities from NGINX can be used for that.
For example there could be multiple servers for receiving and transcoding streams, storing them on some kind of network storage and on the other end multiple NGINX server publishing those streams.

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

### View

```bash
# MPEG-DASH
ffplay http://nginx-rtmp.stream.uos.de/live/dash/mystream.mpd
# HLS
ffplay http://nginx-rtmp.stream.uos.de/live/hls/mystream.m3u8
# Records
curl http://nginx-rtmp.stream.uos.de/records/
ffplay http://nginx-rtmp.stream.uos.de/records/mystream-1596797711.flv
```

## SRS

The configuration for the server `srs.stream.uos.de` is done with the playbook [srs.yml](./srs.yml) and is hopefully self-explanatory and sufficiently commented.

The configuration can be found in [srs.conf](./files/srs/srs.conf).
The enabled features and functionalities can best be seen directly in the configuration.

There are not many tutorials about SRS and in some parts there is unfortunately a language barrier.
For documentation I recommend the [Englisch Wiki](https://github.com/ossrs/srs/wiki/v3_EN_Home) and for configuration the [conf](https://github.com/ossrs/srs/tree/3.0release/trunk/conf) directory in the source code which hold plenty of different configuration examples.

SRS is distributed as [binaries](https://ossrs.net/srs.release/releases/download.html) or for [docker](https://github.com/ossrs/srs-docker).
For this installation I chose using the binaries for Centos 7.

Unfortunately it seems like SRS does not actively support adaptive bitrates, but it is possibly by transcoding the streams and write the manifest files yourself,
like described [here](https://github.com/ossrs/srs/issues/1845#issue-654255515).

There is also a problem with MPEG-DASH, which is probably already described, https://github.com/ossrs/srs/issues/1838.
At least `ffplay` does not find the fragments properly, but e.g. `vlc` does work properly.

### Create Stream

```bash
ffmpeg -re -f lavfi -i testsrc=r=25:s=1280x720,drawtext="fontfile=monofonto.ttf: fontsize=96: box=1: boxcolor=black@0.75: boxborderw=5: fontcolor=white: x=(w-text_w)/2: y=50: text='%{gmtime\:%H\\\\\:%M\\\\\:%S}'" -f lavfi -i sine=f=220:b=4 -vcodec libx264 -pix_fmt yuv420p -preset ultrafast -b:v 1500k -acodec aac -b:a 128k -f flv "rtmp://srs.stream.uos.de/live/mystream"
```

### View

```bash
# MPEG-DASH does have problems, but can be viewed with VLC.
# HLS
ffplay http://srs.stream.uos.de/live/mystream.m3u8
# Record
ffplay http://srs.stream.uos.de/live/mystream.1597573155686.mp4
```
