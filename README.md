## Welcome to [rig-obs](https://github.com/Vigrond/rig-obs)

This [page](https://vigrond.github.io/rig-obs/) will serve as a basic guide to get your own rig-obs project going.

* `rig-obs` simply allows you to setup a streaming rig away from your main rig.  This setup is commonly called a Dual PC streaming setup, but the phrase isn't accurate in this case because with rig-obs you can use a wide array of devices - not just PCs.
* `rig-obs` is simply a technical guide, no code, plugin, or software outside of OBS and ffmpeg is needed.
* `rig-obs` has been shown to work on smartphones such as the Pixel 4a.  It can work with any device that can run OBS Studio or ffmpeg including Windows, OS X, Linux, Android, and IOS.

See the demo below for a quick video showing a Pixel 4a encoding an Overwatch stream to Twitch.tv.  Note the near 0 CPU usage of OBS Studio.

### Disclaimer

This is still a work in progress.  This documentation mostly assumes:

* an Android device as your streaming rig and Twitch as your streaming service, as that is the only thing I have tested.
* You are somewhat familiar with OBS, ffmpeg, and Twitch streaming.

### Demo

* Google Pixel 4a
* 720p @ 48fps
* Low latency and twitch stable
* < 1% CPU usage on OBS
* Works over WiFi

[See this link for a basic demo](https://streamable.com/28gcyw)

### Technology

* OBS Studio
* ffmpeg
* hevc/h265, h264

### Tested Devices

See https://github.com/Vigrond/rig-obs/blob/gh-pages/tested_devices.txt

Have you tested a device?  Please see "Feedback" at the bottom of this document

### General Idea

Use h265/hevc to efficiently get your OBS stream off your main rig, and have another device encode it for your streaming service.

A TCP server is setup on your streaming rig to accept your main rig's OBS stream that is efficiently encoded with hevc/h265 with minimal resource usage.

Once it begins recieving, it can then begin encoding to h264 for popular streaming services like Twitch.

### Preparing your Streaming Rig

Your streaming rig can be a smartphone, a PC, a mac, an Ubuntu box, or anything that can run ffmpeg or OBS.

#### Android device

For this example, we'll setup ffmpeg on an Android device.

- Download and install [Termux](https://github.com/termux/termux-app) from the Play store, or the F Droid store.
- If you want to easily execute the stream server without having to copy paste commands often, I highly recommend using the [F Droid store](https://www.f-droid.org/) to download Termux and Termux Widget apps which are both free in F Droid.  See https://github.com/Vigrond/rig-obs/blob/gh-pages/termux_automation.txt for setup.
- Run `pkg install ffmpeg`
- Take note of your IP4 Address (Settings / About Phone).  Consider setting WiFi to Static IP instead of DHCP.
- Ensure phone has a power source and Battery Saver is turned off.


#### IOS Device

Untested.  If you would like to contribute to this documentation, please open an issue or discussion on this project.

Consider https://ish.app/ for a terminal app on IOS.

#### PC, Mac, Linux

*  Install ffmpeg or OBS Studio

If you are using OBS Studio on your streaming rig, you can ignore the ffmpeg command below and instead use a Media Source, with `local file` turned off and the `tcp://` url discussed in the OBS Setup section as the input.  You are then free to use libx264 or NVENC h264 for encoding to Twitch using the OBS interface.

### Twitch Example

#### Streaming Rig ffmpeg command

This should be ran **first** so the server is ready before OBS begins recording.

* 720p @ 48fps
* IP staticly set to 192.168.1.252 on rig
* Assumes network performance capable of 6000 bitrate
* Assumes 8 core processor for 16 threads 
* Assumes an Android device with ARM processor

This ffmpeg command starts a TCP server that will listen for OBS from your main rig.  Once it begins recieving, it will then encode to x264 and stream to the Twitch ingest server.

Consider using Google Keep for easy copy-paste into the terminal if you want to fiddle with these settings from a main rig.  (Alternative solutions being sought here, perhaps a bash script)

Full command for easy copy paste (Don't forget to put your ingest and streamkey in):

```
ffmpeg -loglevel warning -r 48 -fflags nobuffer -i "tcp://192.168.1.252:9000?listen=1" -c:v libx264 -x264opts keyint=96:no-scenecut -b:v 6000k -minrate 6000k -maxrate 6000k -bufsize 4500k  -profile:v baseline -tune zerolatency -frame_drop_threshold 5.0 -preset superfast -threads 16 -g 96 -keyint_min 48 -c:a copy -bsf:a aac_adtstoasc -flvflags no_duration_filesize -f flv "rtmp://{ingest}.contribute.live-video.net/app/{stream_key}"
```

| Setting                                                  |  Notes         |
| -------------                                          |  ------------- |
| `-loglevel warning`   |  If you want lots of output, use `debug` instead  |
| `-r 48`         | Keep the rate at 48fps when decoding our hevc stream, and when we encode as well.  |
| `-fflags nobuffer`           | We want to keep latency low  |
| `-i "tcp://192.168.1.252:9000?listen=1"`           | We start this connection as a server, as indicated by ?listen=1 |
| `-c:v libx264`              | We are going to encode to x264 for Twitch  |
| `-x264opts keyint=96:no-scenecut`          | Keyframe interval every 96 frames, or 2*fps, which is equivelent to every 2 seconds when fps is 48.  |
| `-b:v 6000k -minrate 6000k -maxrate 6000k -bufsize 4500k `             | Constant bitrate set at the maximum our  network can handle for Twitch.  Buffer size can be .5x-2x the rate depending on screen movement |
| `-profile:v baseline -tune zerolatency`           | We want low latency, so we choose baseline without much of the time consuming features, and we set tune to `zerolatency`.  |
| `-frame_drop_threshold 5.0`           | If our frames begin lagging behind too much (more than 5), go ahead and drop them to help keep up. |
| `-preset superfast -threads 16`              | Preset should likely be ultrafast, superfast, or veryfast depending on the power of your processor.  Threads should be 2*cores.  In this case the Android phone has 8 cores.  Remember, ARM Processor cores are not all equal. |
| `-g 96 -keyint_min 48`           | More keyframe settings to maintain 2 per second.  |
| `-c:a copy`           | Just copy the audio stream as is.  No decoding necessary. |
| `-bsf:a aac_adtstoasc`              | Or so we thought.  mpegts container requires we do a bit of work on our audio before sending it  |
| `-flvflags no_duration_filesize`              | Remove unnecessary header info to cleanup debug logs a bit  |
| `-f flv "rtmp://{ingest}.contribute.live-video.net/app/{stream_key}` | Send our output to a Twitch ingest server.  https://stream.twitch.tv/ingests/  This is also where your stream key goes.  |


#### Main Rig OBS Setup

These settings apply to your OBS Studio installation on your main rig.

Once the ffmpeg command on your streaming rig is running and accepting connections, you can then hit "Start Recording" in OBS on your main rig to begin.

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

![image](https://raw.githubusercontent.com/Vigrond/rig-obs/gh-pages/rigobs_output.jpg)

### Things to consider

* The hevc is heavy to decode.  The device must both decode and encode to h264 for twitch digestion.  hevc bitrate matters!
* h264 bitrate should be the max your network can perform, since you do not have to worry about decoding it
* If you find your stream latency lagging behind increasingly over time, your rig-obs device is struggling to decode and encode!

### Troubleshooting

coming soon...

### Ideas for the Future

* Hardware support for device GPUS (Adreno, etc) using [h264_vaapi](https://trac.ffmpeg.org/wiki/Hardware/VAAPI) with a custom built ffmpeg binary.  Rooted device may be required.

### Support or Feedback

Did it work for you?  Please post about your findings in Discussions.

Head over to [Issues](https://github.com/Vigrond/rig-obs/issues) for Support or [Discussions](https://github.com/Vigrond/rig-obs/discussions) for feedback
