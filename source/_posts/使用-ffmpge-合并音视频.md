---
title: 使用 ffmpge 合并音视频
date: 2021-09-18T23:20:23+08:00
updated: 2021-09-19T10:09:35+08:00
tags:
---

https://superuser.com/questions/277642/how-to-merge-audio-and-video-file-in-ffmpeg
https://stackoverflow.com/questions/12938581/ffmpeg-mux-video-and-audio-from-another-video-mapping-issue

<!-- more -->

## Merging video and audio, with audio re-encoding

See this example, taken from [this blog entry](http://crazedmuleproductions.blogspot.com/2005/12/using-ffmpeg-to-combine-audio-and.html) but updated for newer syntax. It should be something to the effect of:

```
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac output.mp4
```

Here, we assume that the video file does not contain any audio stream yet, and that you want to have the same output format (here, MP4) as the input format.

The above command transcodes the audio, since MP4s cannot carry PCM audio streams. You can use any other desired audio codec if you want. See the [FFmpeg Wiki: AAC Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/AAC) for more info.

If your audio or video stream is longer, you can add the `-shortest` option so that ffmpeg will stop encoding once one file ends.

## Copying the audio without re-encoding

If your output container can handle (almost) any codec – like MKV – then you can simply copy both audio and video streams:

```
ffmpeg -i video.mp4 -i audio.wav -c copy output.mkv
```

## Replacing audio stream

If your input video already contains audio, and you want to replace it, you need to tell ffmpeg which audio stream to take:

```
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

The [`-map` option](https://trac.ffmpeg.org/wiki/Map) makes ffmpeg only use the first video stream from the first input and the first audio stream from the second input for the output file.

## Mix video and audio (from another video)

### Overview of inputs

`input_0.mp4` has the desired video stream and `input_1.mp4` has the desired audio stream:

![mapping diagram](使用-ffmpge-合并音视频/mapping_diagram.png)

In `ffmpeg` the streams look like this:

```
$ ffmpeg -i input_0.mp4 -i input_1.mp4

Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'input_0.mp4':
  Duration: 00:01:48.50, start: 0.000000, bitrate: 4144 kb/s
    Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 1280x720, 4014 kb/s, SAR 115:87 DAR 1840:783, 23.98 fps, 23.98 tbr, 16k tbn, 47.95 tbc (default)
    Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 124 kb/s (default)

Input #1, mov,mp4,m4a,3gp,3g2,mj2, from 'input_1.mp4':
  Duration: 00:00:30.05, start: 0.000000, bitrate: 1754 kb/s
    Stream #1:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 720x480 [SAR 8:9 DAR 4:3], 1687 kb/s, 59.94 fps, 59.94 tbr, 60k tbn, 119.88 tbc (default)
    Stream #1:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 55 kb/s (default)
```

### ID numbers

`ffmpeg` refers to input files and streams with index numbers. The format is `input_file_id:input_stream_id`. Since `ffmpeg` starts counting from 0, stream `1:1` refers to the audio from `input_1.mp4`.

### Stream specifiers

This can be enhanced with [stream specifiers](http://ffmpeg.org/ffmpeg.html#Stream-specifiers-1). For example, you can tell `ffmpeg` that you want the first video stream from the first input (`0:v:0`), and the first audio stream from the second input (`1:a:0`). I prefer this method because it's more efficient. Also, it is less prone to accidental mapping because `1:1` can refer to any type of stream, while `2:v:3` only refers to the fourth video stream of the third input file.

### Examples

The [`-map` option](http://ffmpeg.org/ffmpeg.html#Advanced-options) instructs `ffmpeg` what streams you want. To copy the video from `input_0.mp4` and audio from `input_1.mp4`:

```
$ ffmpeg -i input_0.mp4 -i input_1.mp4 -c copy -map 0:0 -map 1:1 -shortest out.mp4
```

This next example will do the same thing:

```
$ ffmpeg -i input_0.mp4 -i input_1.mp4 -c copy -map 0:v:0 -map 1:a:0 -shortest out.mp4
```

+ `-map 0:v:0` can be translated as: from the first input (`0`), select video stream type (`v`), first video stream (`0`)
+ `-map 1:a:0` can be translated as: from the second input (`1`), select audio stream type (`a`), first audio stream (`0`)

### Additional Notes

+ With `-c copy` the streams will be [stream copied](http://ffmpeg.org/ffmpeg.html#Stream-copy), not re-encoded, so there will be no quality loss. If you want to re-encode, see [FFmpeg Wiki: H.264 Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/H.264).
+ The `-shortest` option will cause the output duration to match the duration of the shortest input stream.
+ See the [`-map` option documentation](http://ffmpeg.org/ffmpeg.html#Advanced-options) for more info.
