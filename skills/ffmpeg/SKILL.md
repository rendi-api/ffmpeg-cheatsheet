---
name: ffmpeg
description: >
  Use when the user asks for FFmpeg or FFprobe commands, video/audio conversion,
  trimming, resizing, padding, overlays, subtitles, thumbnails, GIFs, storyboards,
  slideshows, social-media crops, codec settings, CRF/preset tuning, stream mapping,
  or troubleshooting media automation pipelines.
---

# FFmpeg

Use this skill to produce reliable, explainable FFmpeg/FFprobe commands for media automation.

## Workflow

1. Identify the user's source media, desired output container/codec, target dimensions/duration, and whether quality, speed, or file size matters most.
2. Prefer the simplest command that meets the goal.
3. Use `-c copy` only when no filtering, re-encoding, precise trimming, subtitle burn-in, compression, or codec change is needed.
4. Use explicit stream mapping when multiple inputs or outputs are involved.
5. For commands with filters, quote the filter graph and name streams in `filter_complex` when it improves readability.
6. Include `-y` only when the user wants non-interactive overwrite behavior.
7. Validate command intent with `ffprobe` or a low-duration sample when practical.

## Defaults

- Web-compatible MP4: `-c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p -c:a aac -movflags +faststart`
- High-quality archival H.264: `-c:v libx264 -crf 17` or `-crf 18`
- Smaller modern MP4: `-c:v libx265 -crf 24 -vtag hvc1 -c:a aac`
- WebM/VP9: `-c:v libvpx-vp9 -crf 15 -b:v 0 -c:a libopus`
- Preserve audio when safe: `-c:a copy`
- Force square pixels after scale/pad/crop: `setsar=1:1`
- Playback compatibility from generated images: `format=yuv420p` or `-pix_fmt yuv420p`

## Common command patterns

### Inspect media

```bash
ffprobe -hide_banner -show_streams -show_format input.mp4
```

### Remux without re-encoding

```bash
ffmpeg -i input.mp4 -c copy output.mkv
```

### Resize and pad to vertical 1080x1920

```bash
ffmpeg -i input.mp4 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:color=black,setsar=1:1" \
  -c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p -c:a aac -movflags +faststart \
  output.mp4
```

### Frame-accurate trim

Use output seeking and re-encode for reliable exact cuts:

```bash
ffmpeg -i input.mp4 -ss 00:00:10 -to 00:00:25 \
  -c:v libx264 -crf 18 -preset slow -c:a aac output_trimmed.mp4
```

Use input seeking before `-i` only when speed matters more than accuracy.

### Replace audio

```bash
ffmpeg -i input.mp4 -i music.mp3 \
  -map 0:v -map 1:a -shortest -c:v copy -c:a aac output.mp4
```

### Mix background music under original audio

```bash
ffmpeg -i input.mp4 -i music.mp3 \
  -filter_complex "[1:a]volume=0.2[bgm];[0:a][bgm]amix=inputs=2:duration=shortest[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -shortest output.mp4
```

### Overlay logo for a time range

```bash
ffmpeg -i input.mp4 -i logo.png \
  -filter_complex "overlay=x=(main_w-overlay_w)/8:y=(main_h-overlay_h)/8:enable='gte(t,1)*lte(t,7)'" \
  -c:v libx264 -crf 18 -preset slow -c:a copy output.mp4
```

### Burn subtitles

```bash
ffmpeg -i input.mp4 \
  -vf "subtitles=subtitles.srt:fontsdir=.:force_style='FontName=Poppins,FontSize=24,PrimaryColour=&HFFFFFF'" \
  -c:v libx264 -crf 18 -preset slow -c:a copy output_subtitled.mp4
```

### Add soft subtitle track

```bash
ffmpeg -i input.mp4 -i subtitles.srt \
  -c copy -c:s srt -disposition:s:0 default output.mkv
```

### Thumbnail

```bash
ffmpeg -i input.mp4 -ss 00:00:07 -frames:v 1 -q:v 2 thumbnail.jpg
```

### GIF from video

```bash
ffmpeg -i input.mp4 \
  -vf "select='gt(trunc(t/2),trunc(prev_t/2))',setpts='PTS*0.1',scale=trunc(oh*a/2)*2:320:force_original_aspect_ratio=decrease,pad=trunc(oh*a/2)*2:320:-1:-1" \
  -loop 0 -an output.gif
```

Takes every second frame and accelerates playback 10x; `scale=trunc(oh*a/2)*2:320` forces an even width derived from a 320px height while preserving aspect ratio. Use `-loop 1` for play-once.

## Gotchas

- `-c copy` cannot apply filters and is not frame-accurate for most cuts.
- `-ss` before `-i` is fast but approximate; `-ss` after `-i` is slower but accurate.
- MP4 playback in QuickTime and browsers often needs `yuv420p` for H.264 output.
- Use `-movflags +faststart` for web-hosted MP4/MOV/M4A so playback starts sooner.
- In filter graphs, `[0:v]` means first input's video stream, `[1:a]` means second input's audio stream, and `-map` chooses which streams become outputs.
- Escaping filter expressions is shell-sensitive. Prefer single quotes inside double-quoted filter graphs for time expressions, and escape commas inside nested expressions when required.
- `drawtext` with special characters is fragile; prefer `textfile=` for user-supplied text.
- Burned subtitles require re-encoding; soft subtitle tracks can often use stream copy.

## When more depth is needed

Load `references/command-patterns.md` for additional examples and decision rules distilled from this repository's FFmpeg cheat sheet.
