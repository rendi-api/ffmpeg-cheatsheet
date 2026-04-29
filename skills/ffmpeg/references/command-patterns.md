# FFmpeg Command Patterns

Distilled from this repository's FFmpeg cheat sheet.

## Stream and filter notation

- `-vf` / `-filter:v`: video-only filter chain.
- `-af` / `-filter:a`: audio-only filter chain.
- `-filter_complex`: multi-input, multi-output, or mixed audio/video filter graph.
- `[0]`: all streams from first input.
- `[0:v]`: video from first input.
- `[1:a]`: audio from second input.
- `[name]`: named intermediate/output stream in a complex filter.
- `-map [name]`, `-map 0:a`, etc.: choose streams for each output.

## Resize, crop, and pad

Fit inside a target while preserving aspect ratio:

```bash
-vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:color=black,setsar=1:1"
```

Use `h=-2` or `w=-2` when only one dimension is fixed and codec-compatible even dimensions are needed.

Crop values are `crop=width:height:x:y`, where `x:y` is top-left. Use `pad` after crop when crop windows may exceed source boundaries.

## Audio patterns

Extract audio to mono 16 kHz MP3:

```bash
ffmpeg -i input.mp4 -ar 16000 -b:a 48k -codec:a libmp3lame -ac 1 output.mp3
```

Extract AAC without re-encoding:

```bash
ffmpeg -i input.mp4 -map 0:a:0 -c:a copy output.aac
```

Crossfade two MP3 tracks:

```bash
ffmpeg -i first.mp3 -i second.mp3 \
  -filter_complex "[0:0][1:0]acrossfade=d=3:c1=exp:c2=qsin" \
  -c:a libmp3lame -q:a 2 output.mp3
```

## Speed and timing

Change playback speed without pitch distortion:

```bash
ffmpeg -i input.mp4 \
  -filter_complex "[0:v]setpts=PTS/1.5[v];[0:a]atempo=1.5[a]" \
  -map "[v]" -map "[a]" output.mp4
```

Jump cuts with timestamp reset:

```bash
ffmpeg -i input.mp4 \
  -vf "select='between(t,0,5.7)+between(t,11,18)',setpts=N/FRAME_RATE/TB" \
  -af "aselect='between(t,0,5.7)+between(t,11,18)',asetpts=N/SR/TB" \
  output.mp4
```

## Text, subtitles, and overlays

Timed text overlay:

```bash
-vf "drawtext=text='Get ready':x=50:y=100:fontsize=80:fontcolor=black:box=1:boxcolor=#6bb666@0.6:boxborderw=7:enable='gte(t,1)'"
```

Prefer `drawtext=textfile=caption.txt:fontfile=Font.ttf:...` for special characters or generated captions.

ASS/SRT burn-in color uses `&HBBGGRR` or `&HAABBGGRR`, not CSS RGB order.

## Composing media

Intro + main + outro with normalized streams usually requires matching fps, pixel format, SAR, and audio sample format before `concat`:

```text
[0:v]fps=30,format=yuv420p,setsar=1[intro_v];
[0:a]aformat=sample_fmts=fltp:channel_layouts=stereo[intro_a];
...
[intro_v][intro_a][main_v][main_a][outro_v][outro_a]concat=n=3:v=1:a=1[outv][outa]
```

Stack videos vertically:

```bash
ffmpeg -i top.mp4 -i bottom.mp4 \
  -filter_complex "[0:v]scale=720:-2:force_original_aspect_ratio=decrease,pad=720:640:(ow-iw)/2:(oh-ih)/2:black[top];[1:v]scale=720:-2:force_original_aspect_ratio=decrease,pad=720:640:(ow-iw)/2:(oh-ih)/2:black[bottom];[top][bottom]vstack=inputs=2:shortest=1[v]" \
  -map "[v]" -map 1:a -c:v libx264 -c:a aac -shortest output.mp4
```

## Image/video asset generation

Single image plus audio to video:

```bash
ffmpeg -loop 1 -t 10 -i image.png -i audio.mp3 \
  -vf "scale=1280:720:force_original_aspect_ratio=decrease,pad=1280:720:-1:-1:color=black,setsar=1,fade=t=in:st=0:d=1,format=yuv420p" \
  -c:v libx264 -c:a aac -shortest output.mp4
```

Slideshow with xfade:

```bash
ffmpeg -loop 1 -t 5 -i img1.png -loop 1 -t 5 -i img2.png -i music.mp3 \
  -filter_complex "[0:v]format=yuv420p,fade=t=in:st=0:d=0.5,setpts=PTS-STARTPTS[v0];[1:v]format=yuv420p,fade=t=out:st=4.5:d=0.5,setpts=PTS-STARTPTS[v1];[v0][v1]xfade=transition=fade:duration=0.5:offset=4.5,format=yuv420p[v]" \
  -map "[v]" -map 2:a -c:v libx264 -c:a aac -shortest output.mp4
```

## Thumbnails and storyboards

Scene-change thumbnail:

```bash
ffmpeg -i input.mp4 -vf "select='gt(scene,0.4)'" -frames:v 1 -q:v 2 thumbnail_scene.jpg
```

Storyboard tile from scene changes:

```bash
ffmpeg -i input.mp4 -vf "select='gt(scene,0.4)',scale=640:480,tile=2x2" -frames:v 1 storyboard.jpg
```

Extract keyframes into tiled images:

```bash
ffmpeg -skip_frame nokey -i input.mp4 -vf 'scale=640:480,tile=4x4' -an -vsync 0 keyframes%03d.png
```

For newer FFmpeg versions, prefer `-fps_mode` over deprecated `-vsync` when applicable.

## Encoding decisions

- H.264/libx264: most compatible. Use CRF 17-23; lower is better quality/larger.
- H.265/libx265: smaller files, less universal. Add `-vtag hvc1` for Apple compatibility.
- VP9/libvpx-vp9: web-friendly WebM. Use `-crf N -b:v 0`; recommended CRF often 15-35.
- `-preset` trades encode speed for compression efficiency. `veryslow` compresses better; `ultrafast` encodes faster with larger files.
- `-tune fastdecode` can make playback easier on weak devices.
- `-threads 0` lets FFmpeg choose thread count and is usually default.

## Hardware encoding

NVIDIA:

```bash
ffmpeg -i input.avi -c:v h264_nvenc output.mp4
ffmpeg -i input.avi -c:v hevc_nvenc output.mp4
```

Intel Quick Sync:

```bash
ffmpeg -init_hw_device qsv=hw -filter_hw_device hw -i input.avi -c:v h264_qsv output.mp4
```

GPU encoders are faster but may be less compression-efficient than tuned CPU encoders.
