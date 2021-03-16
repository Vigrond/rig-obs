## Welcome to [rig-obs](https://github.com/Vigrond/rig-ob)

This page will serve as a basic guide to get your own rig-obs project going.

Using native OBS and ffmpeg, you can have your own streaming rig setup on another PC, a smartphone, or any device that can run ffmpeg.

NDI not required!

### Demo

* Pixel 4A
* 720p @ 48fps
* Low latency and twitch stable
* < 1% CPU usage on OBS
* Works over WiFi

[See this link for a basic demo](https://streamable.com/28gcyw)

### Technology

* OBS Studio
* ffmpeg
* hevc/h265, h264

### Twitch Example

#### Rig ffmpeg settings

This should be ran first so the "listener" is setup before OBS begins recording.

* 720p @ 48fps
* IP staticly set to 192.168.1.252 on rig
* Assumes network performance capable of 6000 bitrate
* Assumes 8 core processor for 16 threads

`ffmpeg -loglevel warning -r 48 -fflags nobuffer -i "tcp://192.168.1.252:9000?listen=1" -c:v libx264 -x264opts keyint=96:no-scenecut -r 48 -b:v 6000k -minrate 6000k -maxrate 6000k -bufsize 4500k  -profile:v baseline -tune zerolatency -frame_drop_threshold 5.0 -preset superfast -threads 16 -g 96 -keyint_min 48 -c:a copy -bsf:a aac_adtstoasc -flvflags no_duration_filesize -f flv " rtmp://{ingest}.contribute.live-video.net/app/{stream_key}"`

#### OBS Setup

* Assumes NVIDIA graphics card with hevc_nvenc support

Output / Recording

`Type` Custom Output (FFmpeg)
`FFmpeg Output Type` Output to URL
`File path or URL` tcp://192.168.1.252:9000
`Container format` mpegts


### Support or Feedback

Head over to [Issues](https://github.com/Vigrond/rig-obs/issues) for Support or [Discussions](https://github.com/Vigrond/rig-obs/discussions) for feedback
