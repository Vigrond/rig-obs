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
* Assumes an Android device with ARM processor

```
ffmpeg -loglevel warning -r 48 -fflags nobuffer -i "tcp://192.168.1.252:9000?listen=1" -c:v libx264 -x264opts keyint=96:no-scenecut -r 48 -b:v 6000k -minrate 6000k -maxrate 6000k -bufsize 4500k  -profile:v baseline -tune zerolatency -frame_drop_threshold 5.0 -preset superfast -threads 16 -g 96 -keyint_min 48 -c:a copy -bsf:a aac_adtstoasc -flvflags no_duration_filesize -f flv " rtmp://{ingest}.contribute.live-video.net/app/{stream_key}"
```

#### OBS Setup

* Assumes NVIDIA graphics card with hevc_nvenc support
* Assumes an Android device with ARM processor

##### Output / Recording

| Setting       | Value                     |  Notes         |
| ------------- | -------------             |  ------------- |
| Type          | Custom Output (FFmpeg)    |  We will use ffmpeg  |
| Output Type   | Output to URL             |  We will stream directly to our device  |

```
Type                    Custom Output (FFmpeg)
FFmpeg                  Output Type Output to URL
File path or URL        tcp://192.168.1.252:9000
Container format        mpegts
Video Bitrate           4500Kbps
Keyframe interval       250
Video Encoder           hevc_nvenc
Video Encoder Settings  preset=7 zerolatency=1 profile=rext
Audio Encoder           aac
```
##### Audio

```
Sample Rate             44.1 kHz
```

##### Video

```
Output Resolution       1280x720
Downscale Filter        Lanczos
Common FPS              48
```

##### Advanced

```
Color Format            NV12
```

### Things to consider

* The hevc is heavy to decode.  The device must both decode and encode to h264 for twitch digestion.  hevc bitrate matters!
* h264 bitrate should be the max your network can perform, since you do not have to worry about decoding it
* If you find your stream latency lagging behind increasingly over time, your rig-obs device is struggling to decode and encode!

### Support or Feedback

Head over to [Issues](https://github.com/Vigrond/rig-obs/issues) for Support or [Discussions](https://github.com/Vigrond/rig-obs/discussions) for feedback
