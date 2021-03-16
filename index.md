## Welcome to [rig-obs](https://github.com/Vigrond/rig-obs)

This [page](https://vigrond.github.io/rig-obs/) will serve as a basic guide to get your own rig-obs project going.

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

| Setting       | Value                                                   |  Notes         |
| ------------- | -------------                                           |  ------------- |
|  **Output / Recording**      |                                          |                |
| `Type`                       | *Custom Output (FFmpeg)*                 |  We will use ffmpeg  |
| `FFmpeg Output Type`         | *Output to URL*                          |  We will stream directly to our device  |
| `File path or URL`           | *tcp://192.168.1.252:9000*               |  Need TCP since we will be sending data over WiFi  |
| `Container format`           | *mpegts*                                 |  mpegts supports hevc/h265  |
| `Video Bitrate`              | *4500Kbps*                               |  hevc is heavy to decode, we must be gentle here.  Try about 70% of your twitch bitrate first.  |
| `Keyframe interval`          | *250*                                    |  We want high compression, so we choose a high I-frame interval  |
| `Video Encoder `             | *hevc_nvenc*                             |  Our hevc encoder that makes this possible  |
| `Video Encoder Settings`     | *preset=7 zerolatency=1 profile=rext*    |  We want low latency.  Rext profile is better for our purpose https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=7265015, preset reference: https://github.com/Vigrond/rig-obs/blob/gh-pages/hevc_nvenc_options.txt      |
| `Audio Encoder`              | *aac*                                    |  Choosing `aac` here will let us skip decoding audio and just copy  |
|  **Audio**                   |                                          |                |
| `Sample Rate`                | *44.1 kHz*                               |  Lower CPU Usage, not super necessary.  |
|  **Video**                   |                                          |                |
| `Output Resolution`          | *1280x720*                               |  720p.  We want to use the efficiency of OBS and hevc while we can.  |
| `Downscale Filter`           | *Lanczos*                                |  Lanczos.  |
| `Common FPS`                 | *48*                                     |  While a higher value could theoretically improve latency, we want to make it easy on the decoder.  |
|  **Advanced**                   |                                          |                |
| `Color Format`                | *NV12*                               |  NV12.  |

https://raw.githubusercontent.com/Vigrond/rig-obs/gh-pages/rigobs_output.jpg

### Things to consider

* The hevc is heavy to decode.  The device must both decode and encode to h264 for twitch digestion.  hevc bitrate matters!
* h264 bitrate should be the max your network can perform, since you do not have to worry about decoding it
* If you find your stream latency lagging behind increasingly over time, your rig-obs device is struggling to decode and encode!

### Support or Feedback

Head over to [Issues](https://github.com/Vigrond/rig-obs/issues) for Support or [Discussions](https://github.com/Vigrond/rig-obs/discussions) for feedback
