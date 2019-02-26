# ffmpeg-cheatsheet
Cheat sheet for ffmpeg.
# ffmpeg 
## understanding & examples

```
ffmpeg -i input_file […parameter list…] output_file
ffmpeg <global-options> <input-options> -i <input> <output-options> <output>

```
{Input_file,output_file} <=>  {file, http, pipe, rtp/rtsp, raw udp, rtmp for all REGEX}
- Supply '-f' in  case your input file type is different from a normal file/rtsp stream.
ex- grabbing screen, "images" as input regex.
```
ffmpeg -f image2 -i image%d.jpg –r 25 video.flv
```
- ffmpeg calls the FFmpeg application in the command line window, could also be the full path to the FFmpeg binary or .exe file
- `-i` is follwed by the path to the input video
- `-c:v` sets the video codec you want to use
- Options include libx264 for H.264, libx265 for H.265/HEVC, libvpx-vp9 for VP9, and copy if you want to preserve the video codec of the input video
- `-b:v` sets the video bitrate, use a number followed by M to set value in Mbit/s, or K to set value in Kbit/s
- `-c:a` sets the audio codec you want to use Options include aac for use in combination with H.264 and H.265/HEVC, libvorbis for VP9, and copy if you want to preserve the audio codec of the input video
- `-b:a` sets the audio bitrate of the output video
- `-vf` sets so called video filters, which allow you to apply transformations on a video like scale for changing the resolution and setdar for setting an aspect ratio
- `-r` sets the frame rate of the output video
- `-pix_fmt` sets the pixel format of the output video, required for some input files and so recommended to always use and set to yuv420p for playback
- `-map` allows you to specify streams inside a file
- `-ss` seeks to the given timestamp in the format HH:MM:SS
- `-t` sets the time or duration of the output

![ffmpeg_arch.png](https://github.com/beyondszine/ffmpeg-cheatsheet/blob/master/ffmpeg_arch.png?raw=true)

`SPEED` vs. `QUALITY` vs. `FILE SIZE`
(Lossy) encoding is always a trade-off between:
<p align="center">
  <img src="https://github.com/beyondszine/ffmpeg-cheatsheet/blob/master/ffmpeg_tradeoff.png?raw=true">
</p>

- You can have fast, high-quality encoding, but the file will be large
- You can have high-quality, smaller file size, but the encoding will take longer
- You can have small files with fast encoding, but the quality will be bad.

```sh
ffmpeg -i <input> -c:v libx264 -crf 23 -preset ultrafast -an output.mkv
ffmpeg -i <input> -c:v libx264 -crf 23 -preset medium -an output.mkv
ffmpeg -i <input> -c:v libx264 -crf 23 -preset veryslow -an output.mkv
```

Transcoding: Conversion from one encoder to another.  Ex- H264 to H265
Transmuxing: Conversion from one format/Container to another.  Ex- mp4 to mkv
```sh
# Transcoding from one codec to another (e.g. H.264 using libx264):
ffmpeg -i <input> -c:v libx264 output.mp4
# Transmuxing from one container/format to another – without re-encoding:
ffmpeg -i input.mp4 -c copy output.mkv
```


### Record Screen
```sh
# just video
ffmpeg -video_size 1024x768 -framerate 25 -f x11grab -i :0 output.mp4
# for audio too, co-ordinates are passed along with screen number, i.e. 100,200.
ffmpeg -video_size 1024x768 -framerate 25 -f x11grab -i :0.0+100,200 -f alsa -ac 2 -i hw:0 output.mkv
```
### Remove audio/video
```sh
# disable audio
ffmpeg -i ibiza.mp4 -an ibiza_muted.mp4 # transcoding will be done if codecs differ.
ffmpeg -stats -i ibiza.mp4 -vn -c:a copy ibiza_novid.<container>
# disable video
ffmpeg -i ibiza.mp4 -vn ibiza_novid.mp3 
```

### Mux audio + video
```sh
ffmpeg -i audio.mp4 -i video.mp4 output.mp4
```
### Grab webcam & dump
```sh
ffmpeg -f v4l2 -i /dev/video0 out.mp4
```
You can use the -ss option to specify a start timestamp, and the -t option to specify the encoding duration. The timestamps need to be in HH:MM:SS.xxx format or in seconds (s.msec).

The following would clip the first 30 seconds, and then clip everything that is 10 seconds after that:
```sh
ffmpeg -ss 00:00:30.0 -i input.wmv -c copy -t 00:00:10.0 output.wmv
ffmpeg -ss 30 -i input.wmv -c copy -t 10 output.wmv
```

### Generate random noise
```sh
ffmpeg -ar 48000 -t 60 -f s16le -acodec pcm_s16le -i /dev/u­random -ab 64K -f mp2 -acodec mp2 -y noise.mp2
```

### Add album cover
```sh
ffmpeg -i input.mp3 -i cover.png -c copy -metadata:s:v title="Album cover" -metadata:s:v comment="Cover (Front)" out.mp3
```

### converting file to formats
```sh
ffmpeg -i MyMovie.mkv -vf scale=-1:720 -c:v libx264 -crf 0 -preset veryslow -c:a copy MyMovie_720p.mkv
ffmpeg -i MyMovie.mkv -vf scale=-1:720 -c:v libx264 -crf 18 -preset veryslow -c:a copy MyMovie_720p.mkv
```
The scale video filter is for resizing the video. You just set one size – which is the height in this example – and use -1 for the other dimension. ffmpeg will recalculate the correct value automatically while preserving the aspect ratio.

Srcs:
- [FFmpeg – the swiss army knife of Internet Streaming – part II \| Video Encoding & Streaming Technologies](https://sonnati.wordpress.com/2011/08/08/ffmpeg-%E2%80%93-the-swiss-army-knife-of-internet-streaming-%E2%80%93-part-ii/)
- [Werner Robitza](https://slhck.info/)
- 
