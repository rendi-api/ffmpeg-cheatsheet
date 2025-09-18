# FFmpeg Cheatsheet for Video Automation

A categorized collection of FFmpeg commands for video automation pipelines.

Use this as inspiration for your own work, to troubleshoot your FFmpeg commands, or to explore what others are building in automated media apps.

## How to use this cheatsheet?

- Use the table of contents, best viewed in github's full view of the MD file, to browse through the main topics covered.

- Use Ctrl + F to find what you need. All commands, filters and flags are explained throughout the document. If you don't see a specific explanation, it means that it appears somewhere else in this document and you can search for it (and if you don't find it please open an issue and I will take care of it).

- All sample commands can immediately run from your local machine, since they use sample files that are stored online and FFmpeg is able to download them locally.

- I have attached the original reference to each command, filter, flag, keyword, and explanation. Use these to dig deeper into FFmpeg, and in general - video formats. Some of the findings are my own, in which case no reference was specified.

- There are dropdown arrows used to save on space, you can click on them to get deeper details. The browser is able to search through the text in these hidden sections and reveal them when needed.

- **To make the most of this cheatsheet, it is best to use it along with your favorite LLM (or MCP server or AI agent).** A few ways of doing that:
  - Just copy the full text intro your LLM and let the LLM index of all the information found here. Make sure to copy the RAW version of the MD file for best results.
  - You can also just refer the LLM to the URL of this file and have it index it.
  - If you're interested in a specific command - copy it into the LLM and chat with the LLM to alter this command according to your specific needs
  - You should find all the explanations you require for all the commands within this document and the references it provides. Still, you can always copy a command into the chat interface and have the LLM elaborate.

As [someone on reddit said](https://www.reddit.com/r/ffmpeg/comments/1kdvimj/comment/mqf5dcz):

> "I know it burns a tree every time you ask gpt a question, but it beats slogging through 10 year old answers on stackexchange"

## About LLMs and FFmpeg

I used LLMs as much as I could to make the work for this file as easy as possible, still, all commands and explanations have been tested and vetted by me manually. Many of them I have used in the pre-GPT era - hinting at how old I'm getting (🙈)

LLMs miss out on FFmpeg because it sometimes requires accuracy and attention to fine details that are hard to find online, especially when working with complex filters. I like to use it as a sophisticated search and summarization engine - pointing out specific details and keywords that I then validate online.

🛠️ - Headlines marked with 🛠️ are ones that were especially hard to find correct solutions or explanations with LLMs, or are too important to trust LLMs with the info, so I did manual research and trial and error.

## Glossary of common flags\filters

For those looking to optimize their existing FFmpeg commands, skip to the section starting at [Command settings](#command-settings)

[Filtering](https://ffmpeg.org/ffmpeg-filters.html#Filtering-Introduction)

- `-vf` (also `-filter:v`) video filter
- `-af` (also `-filter:a`) audio filter
- `filter_complex` Complex filter graph - used for general filtering, controlling both audio and video across all streams

Common filter keywords (you can change the numbers to specify the required index):

- `[0]` Select all streams from the first input (0-based index)
- `[0:v]` Select the video stream from first input
- `[1:a]` Select the audio stream from second input
- `0:v:0` From first input, first video stream (0-based index)
- `0:a:1` From first input, second audio stream (0-based index)
- `[name]` Select a named stream, usually used with `-filter_complex`

[-map [name]](https://trac.ffmpeg.org/wiki/Map) Selecting stream for output

[Expression evaluations](https://ffmpeg.org/ffmpeg-all.html#Expression-Evaluation) `if` , `lte` , `gte` and more

`-y` Auto-overwrite output files if existing. Add this flag to the beginning of every FFmpeg command to avoid it asking for confirmation of overwriting

## Simple editing

### Converting formats

Remux MP4 to MKV:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -c copy big_buck_bunny_720p_16sec.mkv
```

`-c copy` - [Read below](#️--c-copy)

> _MKV and MP4:_ Both are video containers and can store H264 and H265 encoded videos and AAC and MP3 encoded audio. The video quality itself is not determined by the container format but rather by the video codec used to compress the video data.
>
> MKV can contain several streams of video, while MP4 is a more widely supported on different platforms and devices.

Remux MP4 to MOV:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -c copy big_buck_bunny_720p_16sec.mov
```

Encode MP4 to AVI:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 big_buck_bunny_720p_16sec.avi
```

[More about video encoding below](#️-videoaudio-encoding-codecs-and-bitrate)

### Resizing and padding

🛠️ Upscale the video to 1080X1920 preserving the original aspect ratio and adding black padding to fill in gaps as needed:

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -vf "scale=w=1080:h=1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:color=black,setsar=1:1" output_resized_pad.mp4
```

<details>
<summary>Scale details</summary>

[scale=w=1080:h=1920:force_original_aspect_ratio=decrease](https://trac.ffmpeg.org/wiki/Scaling) resize video to fit inside 1080x1920, will automatically lower output dimensions to be equal or below the specified width and height, while fitting the original aspect ratio of the input. In this case, will down-scale the input to 1080X810, before adding padding.

If you are unsure about the height (or width) required to keep the original aspect ratio, you can specify `scale=w=1080:h=-1` and let FFmpeg pick the correct height, while making sure we keep the original aspect ratio and the maximum width is 1080.

Specifying -2 `scale=w=1080:h=-2` forces to use dimension sizes that are divisible by 2

_Notice that we can't use `scale=w=-1:1920` here because it will make FFmpeg pick a width which is larger than 1080, conflicting with the output width we are looking for which is 1080 - resulting in an error._

[force_original_aspect_ratio:](https://trac.ffmpeg.org/wiki/Scaling#fit)

> ͏ Achievable with "force_original_aspect_ratio". Of 3 possible values:\
>  ͏ |0| "disable" (default)\
>  ͏ |1| "decrease": auto-decrease output dimensions on need.\
>  ͏ |2| "increase": auto-increase output dimensions on need.

</details>

<details>
<summary>Pad details</summary>

[pad=1080:1920:(ow-iw)/2:(oh-ih)/2:color=black](https://ffmpeg.org/ffmpeg-filters.html#pad) Center the resized video and fill the rest with black padding. Values are `width:height:x:y` where x:y is the top left corner. Negative values also place the image at the center, so you can use `pad=1080:1920:-1:-1:color=black` for a similar effect.

[setsar=1:1](https://ffmpeg.org/ffmpeg-filters.html#setdar_002c-setsar) Sample aspect ratio - ensures the output pixels scale exactly to 1x1 per pixel. It could also be set to `1` or `1/1` - these are all the same. In some cases, FFmpeg may set the Sample Aspect Ratio to compensate for ratio change. Expressly state SAR 1:1 to make things work intended.

</details>

Create two scaled videos from the same input video using one FFmpeg command - one horizontal and another vertical. To the vertical video add an overlay\logo to the top:

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -i https://storage.rendi.dev/sample/rendi_banner_white.png -filter_complex "[0:v]split=2[s0][s1];[s0]scale=w=1920:h=1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:color=black,setsar=1:1[out1];[s1]scale=w=720:h=1280:force_original_aspect_ratio=decrease,pad=720:1280:(ow-iw)/2:(oh-ih)/2:color=black,setsar=1:1[s2];[s2][1]overlay=(main_w-overlay_w)/2:(main_w-overlay_w)/5[out2]" -map [out1] -map 0:a output_youtube.mp4 -map [out2] -map 0:a  output_shorts.mp4
```

Two stackoverflow sources of info I constantly use [Link 1](https://superuser.com/questions/547296/resizing-videos-with-ffmpeg-avconv-to-fit-into-static-sized-player/547406#547406) ; [Link 2](https://superuser.com/questions/991371/ffmpeg-scale-and-pad)

### Trim by time

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -ss 00:00:10 -to 00:00:25 output_trimmed.mp4
```

There are faster ways to trim, but they are less accurate or can create black frames.

[For the advanced explanation see input\output seeking below](#️-inputoutput-seeking-tracffmpegorgwikiseeking)

## Audio Processing

### Replace audio in video

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -i https://storage.rendi.dev/sample/Neon_Lights_5sec.mp3 -map 0:v -map 1:a  -shortest -c:v copy -c:a aac output_replace_audio.mp4
```

Replace the audio in the video with a new audio file

[-shortest](https://ffmpeg.org/ffmpeg.html#Advanced-options) Trims the video's end to be as short as the audio. If you want to keep the video length you can remove this flag (and the output will be muted after 5 seconds)

> _Note: Above command is unexpected in that it has `c:v copy` - it trims in places that are not keyframes without re-encoding so I would have expected to see black frames. But, the output video looks perfect. Also, when trying to explicitly re-encode with `-c:v libx264` the output video turned out to be 7 seconds long, longer than the shortest 5 second audio. Searching online I couldn't find an explanation for both these things._

### Extract audio from video

Encode MP4 to MP3:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 big_buck_bunny_720p_16sec.mp3
```

Extract the audio from an MP4 video, downsample it to 16,000 Hz, convert it to mono MP3, also extract the video (muted):

```sh
ffmpeg -i  https://storage.rendi.dev/sample/popeye_talking.mp4 -ar 16000 -ab 48k -codec:a libmp3lame -ac 1 output_extracted_audio.mp3 -map 0:v -c:v copy -an out_video_only.mp4
```

[FFmpeg audio options](https://ffmpeg.org/ffmpeg.html#Audio-Options)

- `-ar` Sample rate 16KHz - the amount of digital audio wave samples per second
- `-b:a 48k` (which is the same as `-ab`) Bitrate 48KBit/s - which is the amount of data stored per second [Stackoverflow reference](https://superuser.com/questions/319542/how-to-specify-audio-and-video-bitrate)
- `-ac 1` audio channels - 1 (mono)

Extract AAC audio from MP4 without encoding it:

```sh
ffmpeg -i  https://storage.rendi.dev/sample/popeye_talking.mp4 -map 0:a:0 -acodec copy output.aac
```

### Mix the audio in video

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -i https://storage.rendi.dev/sample/Neon_Lights_5sec.mp3 -filter_complex "[1:a]volume=0.2[a1];[0:a][a1]amix=inputs=2:duration=shortest" -shortest -map 0:v -c:v copy -c:a aac output_mix_audio.mp4
```

Mix the audio in the video with a new audio file and lower its volume:

[AAC Reference](https://trac.ffmpeg.org/wiki/Encode/AAC)

[[1:a]volume=0.2[a1]](https://trac.ffmpeg.org/wiki/AudioVolume) Lowers the volume of the audio file so we could also hear the audio from the video file, `[1:a]` means audio from file 1, in a 0-based index. `[a1]` marks the changed volume audio so that we could mix it with the video's audio.

[[0:a][a1]amix=inputs=2](https://ffmpeg.org/ffmpeg-filters.html#amix) Takes the audio from the first stream (the video) and the changed volume audio and mixes them together

<details>
<summary>More details</summary>

If you don't want to change volumes, you can just use this filter instead: `-filter_complex "[0:a][1:a]amix=inputs=2:duration=shortest"`

`:duration=shortest` makes the new audio as short as the shortest audio, the next `-shortest` flag is still required because it controls the length of the final output video (and not just its audio)

[Nice discussion](https://superuser.com/questions/801547/ffmpeg-add-audio-but-keep-video-length-the-same-not) about cases when video is shorter or longer than audio and you want to align the output video's length accordingly

[An open bug around this topic](https://trac.ffmpeg.org/ticket/9487) Using `duration:shortest` and `-shortest` avoids the implications of the bug.

</details>

### Combine two mp3 tracks

```sh
ffmpeg -i https://storage.rendi.dev/sample/Neon_Lights_5sec.mp3 -i https://storage.rendi.dev/sample/Neon%20Lights.mp3 -filter_complex "[0:a]afade=t=out:st=2:d=3[a0];[1:a]afade=t=in:st=0:d=3[a1];[a0][a1]concat=n=2:v=0:a=1" -c:a libmp3lame -q:a 2 output_gapless_fade.mp3
```

[[0:a]afade=t=out:st=2:d=3...[1:a]afade=t=in:st=0:d=3](https://ffmpeg.org/ffmpeg-filters.html#afade-1) Fade out the first and fade in the second:

- First input audio `[0:a]` 3-second `d=3` fade out `t=out` starting from its 2nd second `st=2`
- Second audio `[1:a]` 3-second `d=3` fade-in `t=in` at start of the audio `st=0`

[[a0][a1]concat=n=2:v=0:a=1](https://ffmpeg.org/ffmpeg-filters.html#concat) Concatenates the two faded audio streams back together to create 1 output audio stream, no video (`v=0`).

[-q:a 2](https://trac.ffmpeg.org/wiki/Encode/MP3) - High quality audio output with an average stereo bitrate of 170-210 KBit/s

### Crossfade

```sh
ffmpeg -i https://storage.rendi.dev/sample/Neon_Lights_5sec.mp3 -i https://storage.rendi.dev/sample/Neon%20Lights.mp3 -filter_complex "[0:0][1:0]acrossfade=d=3:c1=exp:c2=qsin" -c:a libmp3lame -q:a 2 output.mp3
```

[acrossfade=d=3:c1=exp:c2=qsin](https://ffmpeg.org/ffmpeg-filters.html#acrossfade) 3-second audio crossfade where first track fades out quickly while second track fades in slowly

### Change audio format

MP3 to WAV pcm_s32le (unsigned 32-bit little-endian) format, mono and 48KHz sample frequency:

```sh
ffmpeg -i https://storage.rendi.dev/sample/Neon%20Lights.mp3 -acodec pcm_s32le -ac 1 -ar 48000 output.wav
```

[Reference](https://trac.ffmpeg.org/wiki/audio%20types)

Merge the audio from two mp4 files, mix them into mono equally, normalizes the volume, downsample to 16 kHz, encode as MP3 at 64 KBits/s:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -i https://storage.rendi.dev/sample/popeye_talking.mp4 -filter_complex "[0:a][1:a]amix=inputs=2:duration=longest,pan=mono|c0=.5*c0+.5*c1,dynaudnorm" -ar 16000 -c:a libmp3lame -b:a 64k merged_audio.mp3
```

`pan=mono|c0=.5*c0+.5*c1` The output channel (c0) is made by blending 50% of the left input (`c0`) and 50% of the right input (`c1`).

`dynaudnorm` Applies dynamic audio normalization (smoothens loud/quiet parts)

[FFmpeg docs about panning and stereo to mono](https://trac.ffmpeg.org/wiki/AudioChannelManipulation)

## Advanced editing

### Change playback speed, without distorting audio

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -filter_complex "[0:v]setpts=PTS/1.5[v];[0:a]atempo=1.5[a]" -map "[v]" -map "[a]" output_sped_up.mp4
```

[setpts=PTS/1.5](https://ffmpeg.org/ffmpeg-filters.html#setpts_002c-asetpts) speeds up video by 1.5x. `atempo=1.5` speeds up audio playback rate while preserving pitch

### Change video frame per second without changing audio speed

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -filter:v fps=60 popeye_fps.mp4
```

[Stackoverflow reference](https://stackoverflow.com/questions/45462731/using-ffmpeg-to-change-framerate)

### Jump cuts

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -vf "select='between(t,0.0,5.7)+between(t,11.0,18.0)+between(t,19.0,20.0)',setpts=N/FRAME_RATE/TB" -af "aselect='between(t,0.0,5.7)+between(t,11.0,18.0)+between(t,19.0,20.0)',asetpts=N/SR/TB" popeye_jumpcuts.mp4
```

Used for making clips shorter, silence removal, removing transitions, etc.

[setpts=N/FRAME_RATE/TB...asetpts=N/SR/TB](https://ffmpeg.org/ffmpeg-filters.html#setpts_002c-asetpts) Reset video and audio presentation timestamps according to the trims requested

- `N` The count of consumed frames\audio samples, not including the current frame for audio, starting from 0
- `FRAME_RATE` \ `SR` Video frame rate and audio sample rate
- `TB` The timebase of the input timestamps

[Stackoverflow reference](https://stackoverflow.com/questions/50594412/cut-multiple-parts-of-a-video-with-ffmpeg)

### 🛠️ Video cropping for social media

Crop a 1080X720 video to 720X1080 by cropping chunks of video to 480X720 and upscaling them by 1.5 at specific time frames to create a vertical social media video:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "split=3[1][2][3];[1]trim=0.0:4.5,setpts=PTS-STARTPTS,crop=min(in_w-300\,480):min(in_h-0\,720):300:0,scale=720:1080,setsar=1:1[1];[2]trim=4.5:8.5,setpts=PTS-STARTPTS,crop=min(in_w-500\,480):min(in_h-0\,720):500:0,scale=720:1080,setsar=1:1[2];[3]trim=8.5,setpts=PTS-STARTPTS,crop=min(in_w-400\,480):min(in_h-0\,720):400:0,scale=720:1080,setsar=1:1[3];[1][2][3]concat=n=3:v=1" -c:v libx264 -c:a copy output_cropped.mp4
```

<details>

[split=3[1][2][3]](https://ffmpeg.org/ffmpeg-filters.html#split_002c-asplit) Splits the input video into 3 chunks and names them `[1]` `[2]` `[3]`

[trim=0.0:4.5](https://ffmpeg.org/ffmpeg-filters.html#trim) Each crop chunk is a temporary new video starting from the start time and ending in the end time `[3]trim=8.5` does not specify an end time, so it will end with the video

Resetting timestamps with [setpts=PTS-STARTPTS](https://ffmpeg.org/ffmpeg-filters.html#setpts_002c-asetpts) is required when using trim and concat to make sure that concat works correctly over seemingly separate video streams (the trimmed streams)

[crop=min(in_w-300\,480):min(in_h-0\,720):300:0](https://ffmpeg.org/ffmpeg-filters.html#crop) The values are `width:height:x:y` x,y are the top left corner. The min dimensions ensure FFmpeg won't crop outside the designated size of the output frame, before scaling. The minimum calculations are not required in this scenario, they are there as placeholders in case you will require different dimensions or x,y positioning

</details>

If cropping is outside the boundaries of the frame - the crop will distort the video. In order to handle this, we can use black padding to fill in the gaps:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "split=3[1][2][3];[1]trim=0.0:4.5,setpts=PTS-STARTPTS,crop=min(in_w-1200\,480):min(in_h-0\,720):1200:0,pad=480:720:(ow-iw)/2:(oh-ih)/2:color=black,scale=720:1080,setsar=1:1[1];[2]trim=4.5:8.5,setpts=PTS-STARTPTS,crop=min(in_w-500\,480):min(in_h-0\,720):500:0,pad=480:720:(ow-iw)/2:(oh-ih)/2:color=black,scale=720:1080,setsar=1:1[2];[3]trim=8.5,setpts=PTS-STARTPTS,crop=min(in_w-400\,480):min(in_h-0\,720):400:0,pad=480:720:(ow-iw)/2:(oh-ih)/2:color=black,scale=720:1080,setsar=1:1[3];[1][2][3]concat=n=3:v=1" -c:v libx264 -c:a copy output_cropped.mp4
```

### Overlay text on video

Overlay three different text messages on a video, each appearing at a specific time, with a fade-in alpha effect and a semi-transparent background box.:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "drawtext=text='Get ready':x=50:y=100:fontsize=80:fontcolor=black:alpha='if(gte(t,1)*lte(t,3),(t-1)/2,1)':box=1:boxcolor=#6bb666@0.6:boxborderw=7:enable='gte(t,1)', drawtext=text='Set':x=50:y=200:fontsize=80:fontcolor=black:alpha='if(gte(t,6)*lte(t,10),(t-6)/4,1)':box=1:boxcolor=#6bb666@0.6:boxborderw=7:enable='gte(t,6)', drawtext=text='BOOM!':x=50:y=300:fontsize=80:fontcolor=black:alpha='if(gte(t,10)*lte(t,15),(t-10)/5,1)':box=1:boxcolor=#6bb666@0.6:boxborderw=7:enable='gte(t,10)'" -c:v libx264 output_text_overlay.mp4
```

If you have a locally stored font file, you can specify it using: `fontfile=<path_to_file>`, for example: `drawtext=text='Get ready':x=50:y=100:fontsize=80:fontcolor=black:fontfile=arial.ttf`

<details>
<summary>drawtext details</summary>

[drawtext](https://ffmpeg.org/ffmpeg-filters.html#drawtext-1)

Explanation of the "Get ready" overlay `drawtext=text='Get ready':x=50:y=100:fontsize=80:fontcolor=black:alpha='if(gte(t,1)*lte(t,3),(t-1)/2,1)':box=1:boxcolor=#6bb666@0.6:boxborderw=7:enable='gte(t,1)'`:

- [enable='gte(t,1)'](https://ffmpeg.org/ffmpeg-filters.html#Timeline-editing) Controls when the overlay is visible - greater than or equal to 1 seconds. `*` is the AND operator Display from t = 1s
- `alpha='if(gte(t,1)*lte(t,3),(t-1)/2,1)'` Alpha fades in between t=1 to t=3, at all other times it equals 1 (fully opaque)
- `box=1` draws a background behind the text with 7px padding `boxborderw=7`
- [boxcolor=#6bb666@0.6](https://ffmpeg.org/ffmpeg-utils.html#color-syntax) — greenish background #6bb666 at 60% opacity.
- `x=50:y=100` Top left position of box
</details>

Add text overlay to video from a text file and font file:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "drawtext=textfile=sample_text.txt:fontfile=Poppins-Regular.ttf:x=50:y=100:fontsize=40:fontcolor=black:alpha='if(gte(t,1)*lte(t,5),t-1,1)':box=1:boxcolor=#6bb666@0.6:boxborderw=7:enable='gte(t,1)'" -c:v libx264 output_text_font_file.mp4
```

FFmpeg does not download the files within `textfile=` and `fontfile=` , therefore you need to download the file manually from https://storage.rendi.dev/sample/sample_text.txt and https://storage.rendi.dev/sample/Poppins-Regular.ttf

It is recommended to use textfile instead of specifying the text within the FFmpeg command itself, to avoid issues with special characters that could interfere with the command line syntax.

### 🛠️ Add subtitles to a video

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p.mp4 -ss 00:00 -to 00:40 -vf "subtitles=sample_subtitles.srt:fontsdir=.:force_style='FontName=Poppins,FontSize=24,PrimaryColour=&HFFFFFF,OutlineColour=&H4066B66B,Outline=1,BorderStyle=3'" -c:v libx264  -c:a copy output_subtitles.mp4
```

This command burns subtitles with a custom font - `Poppins`, with custom subtitles style. Notice to use the `FontName` (and not the file name) - you can find it when you open the font file. Also, specify the `fontsdir` which holds the font file.

You can download the Poppins font file from https://storage.rendi.dev/sample/Poppins-Regular.ttf

<details>
<summary>Subtitle details</summary>

Colors are either `&HBBGGRR` - blue, green, red or `&HAABBGGRR` if you want to add alpha channel (transparency) with FF being 100% transparent and 00 is no transparency.

`PrimaryColour` is the font color

🛠️ `OutlineColour=&H4066B66B,Outline=1,BorderStyle=3` Configures the green background (40/FF in HEXA is 25% opaque) and #6bb666 color in RGB. In order to make the background appear you have to set `Outline=1,BorderStyle=3` Stylizing the background is a bit tricky, [this reddit thread](https://www.reddit.com/r/ffmpeg/comments/hhytkm/styling_subtitle/) has useful info.

Official FFmpeg documentation: [How To Burn Subtitles Into Video](https://trac.ffmpeg.org/wiki/HowToBurnSubtitlesIntoVideo) ; [Subtitles filter](https://ffmpeg.org/ffmpeg-filters.html#subtitles-1)

</details>

If you want to really customize your subtitles' appearance, the best option is using the ASS subtitles format. [A good source of info which I use it constantly.](https://hhsprings.bitbucket.io/docs/programming/examples/ffmpeg/subtitle/ass.html)

For pixel-perfect subtitle burning with special effects and unique appearances, it is best to create opaque images outside of any subtitle format and burn images on the video with FFmpeg.

Add a default subtitles srt track to the video and store it in an MKV container, without re-encoding the video, the codec remains H264:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -i sample_subtitles.srt -c copy -c:s srt -disposition:s:0 default big_buck_bunny_720p_16sec.mkv
```

`-c:s srt` Subtitle format is srt

[-disposition:s:0](https://ffmpeg.org/ffmpeg.html#Stream-specifiers-1) default Sets on the default subtitles track

Extract the subtitles from the mkv file:

```sh
ffmpeg -i big_buck_bunny_720p_16sec.mkv -map 0:s:0 subs.srt
```

Extract the subtitles from the mkv file

### Combine media assets

Overlay an image on video - add logo\watermark to video:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -i https://storage.rendi.dev/sample/rendi_banner_white_transparent.png -filter_complex "overlay=x=(main_w-overlay_w)/8:y=(main_h-overlay_h)/8:enable='gte(t,1)*lte(t,7)'" -c:v libx264 -c:a  output_logo.mp4
```

<details>
<summary>The above command puts an overlay with a transparent background on top of the video</summary>

`x=(main_w-overlay_w)/8:y=(main_h-overlay_h)/8` Positions the overlay's top left corner horizontally at 1/8th of the remaining space from the left and top

- `main_w\main_h` is the width and height of the main video
- `overlay_w\overlay_h` is the width and height of the overlay image

`enable='gte(t,1)*lte(t,7)'` Controls when the overlay is visible - greater than or equal to 1 seconds and less than or equal to 7 seconds, `*` is the AND operator

</details>

🛠 If you want FFmpeg to control the overlay's transparency you can use this command:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -i https://storage.rendi.dev/sample/rendi_banner_white.png -filter_complex  "[1:v]format=argb,geq='p(X,Y)':a='0.5*alpha(X,Y)'[v1];[0:v][v1]overlay=x=(main_w-overlay_w)/8:y=(main_h-overlay_h)/8:enable='gte(t,1)*lte(t,7)'" -c:v libx264 -c:a copy output_faded_logo.mp4
```

`[1:v]format=argb,geq=r='r(X,Y)':a='0.5*alpha(X,Y)'[v1]` Creates the transparent logo

<details>
<summary>Details</summary>

- `[1:v]` Selects the video stream from the second input (the logo)
- `format=argb` Converts the image to ARGB format, so it works with overlay images that don't have an alpha channel
- [geq='p(X,Y)'](https://ffmpeg.org/ffmpeg-filters.html#geq) defines the color of the pixel of the logo at point X,Y to be the color from the original image. It is required in order to exactly control the transparency of the pixel
- `a='0.5*alpha(X,Y)'` makes the logo 50% transparent by multiplying the alpha channel by 0.5
- `[v1]` marks this processed logo as a new video stream
</details>

Put video on top of a background image - creating a video in a new resolution and aspect ratio:

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -i https://storage.rendi.dev/sample/evil-frank.png -filter_complex "[1:v][0:v]overlay=(W-w)/2:(H-h)/2" -c:v libx264 -c:a copy output_bg.mp4
```

`[1:v][0:v]` First puts the image (background) and on top puts the video.

`(W-w)/2:(H-h)/2` centers the video horizontally and vertically on the background image by picking the video's top left corner accordingly. W\H are background width and height, w\h are video width and height, the capital letters belong to the first specified stream `[1:v]` and lower case is the second specified stream `[0:v]`. Notice that the order is based on `[1:v][0:v]` and not the order of the input files.

Combine intro main and outro to one video and mix with background music:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_5sec_intro.mp4 -i https://storage.rendi.dev/sample/popeye_talking.mp4 -i https://storage.rendi.dev/sample/big_buck_bunny_720p_5sec_outro.mp4 -i https://storage.rendi.dev/sample/Neon%20Lights.mp3 -filter_complex "[0:v]fps=30,format=yuv420p,setsar=1[intro_v];[1:v]scale=-2:720:force_original_aspect_ratio=decrease,pad=1280:720:(ow-iw)/2:(oh-ih)/2:black,fps=30,format=yuv420p,setsar=1[main_v];[2:v]fps=30,format=yuv420p,setsar=1[outro_v];[0:a]aformat=sample_fmts=fltp:channel_layouts=stereo[intro_a];[1:a]aformat=sample_fmts=fltp:channel_layouts=stereo[main_a];[2:a]aformat=sample_fmts=fltp:channel_layouts=stereo[outro_a];[intro_v][intro_a][main_v][main_a][outro_v][outro_a]concat=n=3:v=1:a=1[combined_video][combined_audio];[3:a]volume=0.1,aformat=sample_fmts=fltp,afade=t=in:ss=0:d=1.5,afade=t=out:st=20:d=2[bgm_faded];[combined_audio][bgm_faded]amix=inputs=2:duration=first:dropout_transition=2[final_audio]" -map "[combined_video]" -map "[final_audio]" -c:v libx264 -c:a aac -shortest intro_main_outro.mp4
```

`duration=first` The output audio stream duration should be the like the input stream (the combined audio), `dropout_transition=2` creates a fade out effect for the shorter audio so that it won't cut off abruptly

[aformat=sample_fmts=fltp](https://ffmpeg.org/ffmpeg.html#Audio-Options) Convert audio format to 32-bit float planar (a commonly used format in FFmpeg), couldn't find good simple sources for it online

🛠️ Stack two videos vertically and keep the audio of the second video:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -i https://storage.rendi.dev/sample/popeye_talking.mp4 -filter_complex "[0:v]scale=720:-2:force_original_aspect_ratio=decrease,pad=720:640:(ow-iw)/2:(oh-ih)/2:black[top];[1:v]scale=720:-2:force_original_aspect_ratio=decrease,pad=720:640:(ow-iw)/2:(oh-ih)/2:black[bottom];[top][bottom]vstack=inputs=2:shortest=1[v]" -map "[v]" -map 1:a -c:v libx264 -c:a aac -shortest output_stacked.mp4
```

[shortest=1](https://ffmpeg.org/ffmpeg-filters.html#Options-for-filters-with-several-inputs-_0028framesync_0029) from the two video streams we follow the shortest when vstacking both. `-shortest` between the output video and the audio from the second input video we pick the shortest.

## Asset generation

### Image to video

Create a 10 second video from a looping input image and audio file, image fades into view:

```sh
ffmpeg -loop 1 -t 10 -i https://storage.rendi.dev/sample/bbb-splash.png -i https://storage.rendi.dev/sample/Neon%20Lights.mp3 -vf "scale=1280:720:force_original_aspect_ratio=decrease,pad=1280:720:-1:-1:color=black,setsar=1,fade=t=in:st=0:d=1,format=yuv420p" -c:v libx264 -c:a aac -shortest output_loop.mp4
```

Above command runs slowly because it is downloading the image frame for every video frame. To make it run faster, download the png locally and run the command with the local file.

`-loop 1` Infinitely loop over the input image. `-t 10` the duration of the input loop to take is 10 seconds, so even though we infinitely loop the input image, we stop after 10 seconds.

[Excellent stackoverflow reference about loops - must read](https://video.stackexchange.com/questions/12905/repeat-loop-input-video-with-ffmpeg)

[fade=t=in:st=0:d=1](https://ffmpeg.org/ffmpeg-filters.html#fade) 1-second (`d=1`) fade-in (`t=in`) at start of the video (`st=0`)

🛠️ Create a slideshow video of 5 seconds per input image and background audio, fading between images:

```sh
ffmpeg -loop 1 -t 5 -i https://storage.rendi.dev/sample/rodents.png -loop 1 -t 5 -i https://storage.rendi.dev/sample/evil-frank.png -i https://storage.rendi.dev/sample/Neon%20Lights.mp3 -filter_complex "[0:v]format=yuv420p,fade=t=in:st=0:d=0.5,setpts=PTS-STARTPTS[v0];[1:v]format=yuv420p,fade=t=out:st=4.5:d=0.5,setpts=PTS-STARTPTS[v1];[v0][v1]xfade=transition=fade:duration=0.5:offset=4.5,format=yuv420p[v]" -map "[v]" -map 2:a -c:v libx264 -c:a aac -shortest slideshow_with_fade.mp4
```

The resulting video is 9.5 seconds because there is an overlap of 0.5 second when fading from the first image to the second image. First image is faded in and last image is faded out.

<details>

`[0:v]...[v0];[1:v]...[v1];[v0][v1]...[v]` First input video stream `[0:v]` is filtered with fade in and its result is marked as `v0`, then second input video stream is filtered and its result is marked as `v1`, then they are concatenated together with xfade and the output video result is marked as `v`

[xfade=transition=fade:duration=0.5:offset=4.5](https://trac.ffmpeg.org/wiki/Xfade) Starts fade out transition of the first image at its 4.5 second, which lasts 0.5 second, while adding the second image during the transition.

</details>

🛠️ Create a Ken Burns style video from images:

```sh
ffmpeg -loop 1 -i https://storage.rendi.dev/sample/rodents.png -loop 1 -i https://storage.rendi.dev/sample/evil-frank.png -i https://storage.rendi.dev/sample/Neon%20Lights.mp3 -filter_complex "[0:v]scale=8000:-1,zoompan=z='zoom+0.005':x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':d=100:s=1920x1080:fps=25,trim=duration=4,format=yuv420p,setpts=PTS-STARTPTS[v0];[1:v]scale=8000:-1,zoompan=z='if(lte(zoom,1.0),1.5,max(zoom-0.005,1.005))':x=0:y='ih/2-(ih/zoom/2)':d=100:s=1920x1080:fps=25,trim=duration=4,format=yuv420p,setpts=PTS-STARTPTS[v1];[v0][v1]xfade=transition=fade:duration=1:offset=3,format=yuv420p[v]" -map "[v]" -map 2:a -c:v libx264 -c:a aac -shortest output_kenburns.mp4
```

<details>
<summary>
Command creates a video from two input images and background audio. Zooms in on the first image's center, plays it for 4 seconds and fade transitions to the next image.
Second image zooms out from its left side while playing it for 4 seconds.
Output is 7 seconds long because of 1 second fade transition between the two image chunks and command shortens the audio to match the video
</summary>

`z='zoom+0.005'` Every new frame generated adds 0.005 zoom to the previous frame, or, zooms in the previous frame by 1.005

`x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)'` Pan to the center of the frame

`d=100:s=1920x1080:fps=25` specifies that the effect will generate 100 frames (`d`), of output resolution `s=1920x1080` and 25 frames per second `fps` which is 4 seconds effect (100 frames divided by 25 fps)

`scale=8000:-1` Used to first upscale the frame and then zoom on it, this avoids a [jitteriness bug](https://trac.ffmpeg.org/ticket/4298) which occurs with zoompan filter, at the cost of more compute time for upscaling. `-1` means to adjust the height so that aspect ratio will be preserved according to 8000px width. Good reads: https://superuser.com/a/1112680/431710 , https://superuser.com/questions/1112617/ffmpeg-smooth-zoompan-with-no-jiggle

[zoompan=z='if(lte(zoom,1.0),1.5,max(zoom-0.005,1.005))'](https://ffmpeg.org/ffmpeg-filters.html#zoompan) This part zooms out of a zoomed-in starting frame. If the zoom factor is less than 1.0 then we set it to 1.5 - this corresponds to the starting frame. Then the command zooms out by 0.005 at each frame until it reaches a zoom factor of 1.005, giving the zoom-out effect, and then stops changing the zoom - keeping it from resetting the zoom out effect.

`trim=duration=4` It was not possible to specify `-t=4` before the input file and keep this image chunk at 4 seconds (like above in create a video from a looping input image). When trying to do that, the first chunk is of a correct length because of `xfade` up to 4 seconds, but the second chunk gets repeated so that the total output video matches the audio's length. I tried different ways of solving it, but nothing helped. This is probably due to the zoompan filter which basically eliminates the purpose of `-t` by specifying the fps and the frames number without specifying a hard maximum cap.
The only thing that worked is to specify the trim duration after the zoompan.

[Ken Burns Effect](https://en.wikipedia.org/wiki/Ken_Burns_effect)

[Blog post about ken burns and FFmpeg](https://mko.re/blog/ken-burns-ffmpeg/):

</details>

### Create GIFs

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "select='gt(trunc(t/2),trunc(prev_t/2))',setpts='PTS*0.1',scale=trunc(oh*a/2)*2:320:force_original_aspect_ratio=decrease,pad=trunc(oh*a/2)*2:320:-1:-1" -loop 0 -an output.gif
```

Create a looping gif from video auto-scaled to 320px width, taking every 2nd frame `gt(trunc(t/2),trunc(prev_t/2))` and accelerating the playing speed by 10 `setpts='PTS*0.1'`

`-loop 0` is the default, and can actually be omitted, stating that the loop is indefinite. To only loops once use `-loop 1`

[Good reference](https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality/556031#556031)

### Turn video frames into a video compilation

Create a video compilation based on single input video which gets split into parts, with fade effects:

```sh
ffmpeg -i https://storage.rendi.dev/sample/BigBuckBunny_320x180.mp4 -filter_complex "[0:v]trim=start=11:end=15,setpts=PTS-STARTPTS,fade=t=in:st=0:d=0.5,fade=t=out:st=3.5:d=0.5[v1];[0:a]atrim=start=11:end=15,asetpts=PTS-STARTPTS,afade=t=in:st=0:d=0.5,afade=t=out:st=3.5:d=0.5[a1];[0:v]trim=start=21:end=25,setpts=PTS-STARTPTS,fade=t=in:st=0:d=0.5,fade=t=out:st=3.5:d=0.5[v2];[0:a]atrim=start=21:end=25,asetpts=PTS-STARTPTS,afade=t=in:st=0:d=0.5,afade=t=out:st=3.5:d=0.5[a2];[v1][a1][v2][a2]concat=n=2:v=1:a=1[outv][outa]" -map "[outv]" -map "[outa]" -c:v libx264 -c:a aac output_fade_in_out.mp4
```

The command takes two segments from the input video (11-15 seconds and 21-25 seconds) applies fade in/out effects to each segment and concatenates both.

<details>

`trim=start=X:end=Y` Cuts video to specified time range, `atrim` - corresponding for audio

`setpts=PTS-STARTPTS` Resets timestamps to start from 0

`fade=t=in:st=0:d=0.5...fade=t=out:st=3.5:d=0.5` See above in creating a slideshow

`afade` See above in audio processing

`concat=n=2:v=1:a=1` Combines two segments with both video and audio

</details>

### Create thumbnails from video

Create a thumbnail from the frame in second 7:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -ss 00:00:07 -frames:v 1 output_thumbnail.png
```

[Reference](https://trac.ffmpeg.org/wiki/Create%20a%20thumbnail%20image%20every%20X%20seconds%20of%20the%20video)

To control the output image quality use `-q:v`:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -ss 00:00:07 -frames:v 1 -q:v 2 output_thumbnail.jpg
```

Values are from 2 to 31, 2 being the best and 31 being the worst. References: [Stackoverflow 1](https://stackoverflow.com/questions/10225403/how-can-i-extract-a-good-quality-jpeg-image-from-a-video-file-with-ffmpeg/10234065#10234065) [Stackoverflow 2](https://stackoverflow.com/questions/64011346/ffmpeg-quality-conversion-options-video-compression)

Creates two thumbnails - one from the first frame after second 5 and one from the first frame after second 15:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -filter_complex "[0:v]split=2[first][second];[first]select='gte(t,5)'[thumb1];[second]select='gte(t,15)'[thumb2]" -map [thumb1] -frames:v 1 output_thumbnail_1.png -map [thumb2] -frames:v 1 output_thumbnail_2.png
```

`-frames:v 1` Output only 1 video frame

Create a thumbnail from the first frame of a scene change:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "select='gt(scene,0.4)'" -frames:v 1  -q:v 2 thumbnail_scene.jpg
```

[gt(scene,0.4)](https://www.ffmpeg.org/ffmpeg-filters.html#select_002c-aselect) parameter determines FFmpeg's sensitivity to changes in frame indicating scene change. The value is from 0 to 1, lower values mean FFmpeg will be more sensitive to scene changes and will recognize more scene changes. Recommended values are from 0.3 to 0.5

[Good stackoverflow discussion about detecting scenes with Ffmpeg](https://stackoverflow.com/questions/35675529/using-ffmpeg-how-to-do-a-scene-change-detection-with-timecode)

### Create an image thumbnail from input images

```sh
ffmpeg -i https://storage.rendi.dev/sample/bbb-splash.png -i https://storage.rendi.dev/sample/rodents.png -i https://storage.rendi.dev/sample/evil-frank.png -filter_complex "[1]scale=640:360,pad=648:368:4:4:black[overlay1];[2]scale=640:360,pad=648:368:4:4:black[overlay2];[0][overlay1]overlay=0:main_h-overlay_h[tmp1];[tmp1][overlay2]overlay=main_w-overlay_w:main_h-overlay_h" -frames:v 1 thumbnail_overlayed.png
```

### Create a storyboard from a video

All commands below extract frames from video to create different storyboards

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "select='gt(scene,0.4)',scale=640:480,tile=2X2" -frames:v 1 scene_storyboard.jpg
```

🛠️ Use `tile=2X2` to create a 2X2 storyboard from the scenes in a video. [Example from FFmpeg's documentation](https://www.ffmpeg.org/ffmpeg-filters.html#Examples-169)

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -vf "select='gt(scene,0.4)'" -vsync 0 scene_storyboard_%03d.jpg
```

Create the same storyboard but with separate image files per scene

🛠️ [-vsync 0](https://www.ffmpeg.org/ffmpeg-filters.html#Examples-126) drops frames that belong to the same scene so there are no duplication. This parameter is complex to use, [good explanation](https://superuser.com/a/1073260/431710)

> vsync is fine to use but is deprecated in newer versions of ffmpeg, `-fps_mode` is the change [Reference](https://superuser.com/questions/1816464/ffmpeg-vsync-deprecated-use-fps-mode) [FFmpeg docs](<https://ffmpeg.org/ffmpeg.html#:~:text=%2Dfps_mode%5B%3Astream_specifier%5D%20parameter%20(output%2Cper%2Dstream)>)

```sh
ffmpeg -skip_frame nokey -i https://storage.rendi.dev/sample/big_buck_bunny_720p.mp4 -vf 'scale=640:480,tile=4x4' -an -vsync 0 keyframes%03d.png
```

Create storyboard with several files that are tiled, base the frames the video's keyframes instead of scenes. [Example is from FFmpeg's documentation](https://www.ffmpeg.org/ffmpeg-filters.html#Examples-127)

`-skip_frame nokey` As the text suggests - skips frames that are not key.

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4  -vf "select=not(mod(n\,10)),scale=640:480,tile=4x2"  -vsync 0 tile_4_2_frames_10_%03d.png
```

Create 4X2 tile files from every 10th frame of a video. To just create images per frame, remove the `,tile4x2` part

## Command settings

### Generic, simple and optimized FFmpeg command for daily use

```sh
ffmpeg -i https://storage.rendi.dev/sample/popeye_talking.mp4 -vf "scale=w=1080:h=1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:color=black,setsar=1:1" -crf 18 -preset veryslow  -threads 0 -tune fastdecode  -movflags +faststart  output_scaled_optimized.mp4
```

This command re-sizes the input video and is good for archiving, streaming (non-live) and playing on many different edge devices. You can usually use the flags in this command for all your FFmpeg commands, unless you have a specific reason not to.

The parameters in this command have different configuration options. Make sure to read through their FFmpeg references. Let me know if you would like me to elaborate about them here.

[-tune fastdecode](https://trac.ffmpeg.org/wiki/Encode/H.264#Tune) Encoded output will require less computational power to decode - good for viewing in many different edge devices. [You can use zerolatency for optimization for fast encoding and low latency streaming](https://superuser.com/questions/564402/explanation-of-x264-tune)

[-preset veryslow](https://trac.ffmpeg.org/wiki/Encode/H.264#Preset) Slower encoding, but with a more compressed output keeping the high quality - good when optimize for web-viewing (VOD, archiving, non-live streaming). If your require very fast encoding, at the cost of larger output file use `ultrafast`.

`-threads 0` Specifies how many system threads to use. Optimal is 0 (and is default), usually it is best to just not use this parameter and let FFmpeg optimize. [But sometimes you want to tweak it, depending on your system and command](https://superuser.com/questions/155305/how-many-threads-does-ffmpeg-use-by-default)

### 🛠️ Video\Audio encoding, codecs and bitrate

[-c -codec](https://ffmpeg.org/ffmpeg.html#Stream-specifiers).

`-c:v` Specifies the video encoder and `-c:a` specifies the audio encoder.

`-c:a aac` AAC encoded audio. This is also the default for FFmpeg, and a good practice to specify.

[-c:a libmp3lame](https://trac.ffmpeg.org/wiki/Encode/MP3) The encoding library for MP3

`-an` Disable audio in the output

---

#### [-c:v libx264](https://trac.ffmpeg.org/wiki/Encode/H.264) - H264 (AVC)

Generally FFmpeg will default to H264 when asking for MP4 output, unless you're not using an FFmpeg build with libx264. It's good practice to always specify the codec.

libx265 - H265 (HEVC), the newer codec, is very similar in behavior and controls. H264 is still the most commonly used.

> Apple devices sometimes have issues with FFmpeg generated H265 videos (for example in iOS Airdop), use `-vtag hvc1` to solve it. [Thanks!](https://gist.github.com/protrolium/e0dbd4bb0f1a396fcb55#convert-h264-to-h265-to-correct-for-ios-airdrop-error-message) [Also related](https://community.bitmovin.com/t/whats-the-difference-between-hvc1-and-hev1-hevc-codec-tags-for-fmp4/101)

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -c:v libx265 -vtag hvc1 -c:a copy output_265.mp4
```

[format=yuv420p](https://trac.ffmpeg.org/wiki/Encode/H.264#Encodingfordumbplayers) H264 YUV planar color format, is used for playback compatibility in most players. Use this flag when transforming images to video and in general when you have playback issues with your output videos, unless you have specific reasons not to.

> You may need to use -vf format=yuv420p (or the alias -pix_fmt yuv420p) for your output to work in QuickTime and most other players. These players only support the YUV planar color space with 4:2:0 chroma subsampling for H.264 video. Otherwise, depending on your source, ffmpeg may output to a pixel format that may be incompatible with these players.

Good info about yuv420p in [this reddit thread](https://www.reddit.com/r/ffmpeg/comments/1gbv2z5/is_formatyuv420p_mandatory/)

`-crf` [Constant Rate Factor (CRF)](https://trac.ffmpeg.org/wiki/Encode/H.264#crf) - It is the default bitrate control option for libx264 and libx265:

> Use this rate control mode if you want to keep the best quality and care less about the file size. This is the recommended rate control mode for most uses.
>
> This method allows the encoder to attempt to achieve a certain output quality for the whole file when output file size is of less importance. This provides maximum compression efficiency with a single pass. By adjusting the so-called quantizer for each frame, it gets the bitrate it needs to keep the requested quality level. The downside is that you can't tell it to get a specific filesize or not go over a specific size or bitrate, which means that this method is not recommended for encoding videos for streaming.
>
> The range of the CRF scale is 0–51, where 0 is lossless (for 8 bit only, for 10 bit use -qp 0), 23 is the default, and 51 is worst quality possible. A lower value generally leads to higher quality, and a subjectively sane range is 17–28. Consider 17 or 18 to be visually lossless or nearly so; it should look the same or nearly the same as the input but it isn't technically lossless.
> The range is exponential, so increasing the CRF value +6 results in roughly half the bitrate / file size, while -6 leads to roughly twice the bitrate.

Common advice is to use `-crf 18` for very high quality H264 output, I found that using `-crf 10` results in better quality video.

Use [-movflags +faststart](https://trac.ffmpeg.org/wiki/Encode/H.264#faststartforwebvideo) to make videos start playing faster online, optimizing for web viewing, by moving metadata to the front of the container:

```sh
ffmpeg  -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -c copy -movflags +faststart big_buck_bunny_720p_16sec_faststart.mp4
```

[YouTube recommend](https://support.google.com/youtube/answer/1722171) uploading MP4 files with faststart to YouTube. They will then re-encode these to VP9.

[Fast start is supported in MP4, M4A and MOV and could take a few seconds to process](https://superuser.com/questions/856025/any-downsides-to-always-using-the-movflags-faststart-parameter) I couldn't find an official place that states that faststart works with libx265, but the following command shows that it does work:

```sh
ffmpeg  -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -c:v libx265 -c:a copy -movflags +faststart big_buck_bunny_720p_16sec_h265_faststart.mp4
```

In order to make sure that `big_buck_bunny_720p_16sec_h265_faststart.mp4` is indeed encoded with moov faststart, run

```sh
ffprobe -v trace -i your_video.mp4
```

and check that towards the beginning there is a line that resembles `[mov,mp4,m4a,3gp,3g2,mj2 @ 0x...] type:'moov' size:... pos:...`

---

#### [libvpx-vp9](https://trac.ffmpeg.org/wiki/Encode/VP9)

It is the VP9 video encoder for ​WebM, an open, royalty-free media file format. VP9 is owned by google, and most videos on YouTube are encoded with it. It is an encoding designed and optimized for static web hosted video. `libvpx-vp9` can save about 20–50% bitrate compared to `libx264` (the default H264 encoder), while retaining the same visual quality.

Constant Quality `-crf` encoding in `libvpx-vp9` - similar to constant rate factor in `libx264`:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4 -c:v libvpx-vp9 -crf 15 -b:v 0 -c:a libopus big_buck_bunny_720p_16sec.webm
```

> To trigger this mode, you must use a combination of `-crf` and `-b:v 0`. Note that `-b:v` MUST be 0. Setting it to anything higher or omitting it entirely will instead invoke the Constrained Quality mode.

> The CRF value can be from 0–63. Lower values mean better quality. Recommended values range from 15–35, with 31 being recommended for 1080p HD video.

`-c:a libopus` The default audio encoder for WebM is libopus, the above command re-encodes the AAC audio in mp4 to opus in webm.

[CPU, speed and multithread controls for vp9](https://trac.ffmpeg.org/wiki/Encode/VP9#speed)

---

VP9, libx264 and libx265 support 1-pass and 2-pass encodings (you can read about these in their respective references). [slhck summarized it well:](https://slhck.info/video/2017/03/01/rate-control.html):

> To summarize, here's what you should do [which bitrate encoding configuration to use], depending on your use case:
>
> - Archival — CRF that gives you the quality you want.
> - Streaming — Two-pass CRF or ABR with VBV-constrained bitrate.
> - Live Streaming — One-pass CRF or ABR with VBV-constrained bitrate, or CBR if you can waste bits.
> - Encoding for Devices — Two-pass ABR, typically.

slhck probably meant [two-pass CRF in VP9](https://trac.ffmpeg.org/wiki/Encode/VP9#twopass) for streaming - the first pass lets libvpx-vp9 calculate the desired measures to encode in higher compression in the second pass for reduced file size while keeping quality. This method is more optimized for web hosted videos.

Good references:

- [Another very good reference by slhck about crf](https://slhck.info/video/2017/02/24/crf-guide.html)
- [Stackoverflow CRF in FFmpeg](https://superuser.com/questions/677576/what-is-crf-used-for-in-ffmpeg)
- [Reddit discussion about CRF](https://www.reddit.com/r/compression/comments/vcuory/how_does_crf_compression_work/)
- [Reddit discussion about CRF VS CQP VS CBR and GPU encoding](https://www.reddit.com/r/obs/comments/fj6ysq/cqp_vs_cbr_recording/)
- [Reddit discussion about CBR and CQP](https://www.reddit.com/r/obs/comments/1e9b63m/should_i_use_cqp_or_cbr/)
- [Reddit discussion about CRF and 2-pass](https://www.reddit.com/r/handbrake/comments/13dckje/is_there_a_way_to_see_what_crf_is_equivalent_to/)

### 🛠️ [-c copy](https://ffmpeg.org/ffmpeg.html#Streamcopy)

Use `-c copy` whenever possible, it re-muxes the video and audio instead of re-encoding which is compute intensive (especially video re-encoding). `-c:v copy` Specifically copies video without re-encoding and `-c:a copy` does the same for audio (and is the same as `-acodec copy`).

_Remuxing involves rewrapping streams into a new container without altering them, unlike transcoding, which changes compression and quality. For example - MP4 can be remuxed to MKV and MOV because they are all containers of H264 codec._

When not to use `-c copy`?

- When applying video filters (scale, overlay, subtitles, trim, fade) or mixing or modifying audio (amix, atempo, volume) - these require re-encoding
- For precise trimming (frame-accurate) -c copy can only cut at keyframes, leading to rough/inaccurate edits.
- Burning subtitles into the video requires re-encoding
- Transcoding between different codecs requires re-encoding
- If you want to compress media

### 🛠️ [Input\Output seeking](https://trac.ffmpeg.org/wiki/Seeking)

Input seeking (`-ss` before input):

```sh
ffmpeg -ss 00:00:03 -i "https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4" -frames:v 1 "input_seeking.jpg"
```

Parses the video by keyframe, making it very fast, but less accurate (in h624 with 25fps there is a keyframe every 10 seconds).

If you trim the video with input seeking, it resets the timestamps of the video to the trimmed version, so when using filters you need to make sure to adhere to the video times after trim.

```sh
ffmpeg -i "https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4" -ss 00:00:03 -frames:v 1 "input_seeking.jpg"
```

Output seeking (`-ss` after input) "decodes but discards input until the timestamps reach position" - it is frame accurate, but can take longer time to process because needs to decode.

**🛠️ When trimming, it is advised to use output seeking <ins>without</ins> `-c:v copy`, re-encoding the output video.**

The reasons being:

- There is an open bug with trimming with input seeking and `-c:v copy`
  [Stackoverflow discussion](https://stackoverflow.com/questions/57450657/ffmpeg-fails-to-trim-beginning-of-clip) -
  [FFmpeg repo bug report](https://trac.ffmpeg.org/ticket/8189) .
  Therefore, it is advised to trim with output seeking (with or without `-c:v copy`).

- When trimming with output seeking with `-c:v copy` you can see black frames in the output video,
  this is due to `c:v copy` copying frames that started after a keyframe, but not the keyframe itself,
  which misses out on the data required to do the frames. Read more in [FFmpeg's trac documentation](https://trac.ffmpeg.org/wiki/Seeking#codec-copy)

This excerpt from [FFmpeg's documentation](https://ffmpeg.org/ffmpeg.html#Main-options) sums it all:

> When used as an input option, seeks in this input file to position.
> Note that in most formats it is not possible to seek exactly, so ffmpeg will seek to the closest seek point before position.
> When transcoding and -accurate_seek is enabled (the default), this extra segment between the seek point and position will be decoded and discarded. When doing stream copy or when -noaccurate_seek is used, it will be preserved.

Nice answer about this issue in [Stackoverflow](https://stackoverflow.com/a/18449609/3530894)

The re-encoded output video could be in a different bitrate, so you might need to adjust the output bitrate accordingly (see below).

### 🛠️ Use GPU for acceleration

Transcode video from AVI to H264 (AVC) using Nvidia GPU:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_stereo.avi -c:v h264_nvenc output_gpu_264.mp4
```

Transcode video from AVI to H265 (HEVC) using Nvidia GPU:

```sh
ffmpeg -i https://storage.rendi.dev/sample/big_buck_bunny_720p_stereo.avi -c:v hevc_nvenc output_gpu_265.mp4
```

[Reference](https://trac.ffmpeg.org/wiki/HWAccelIntro#NVENC)

Transcode using the Intel GPU - [Quick Sync Video (QSV) encoder](https://trac.ffmpeg.org/wiki/Hardware/QuickSync):

```sh
ffmpeg -init_hw_device qsv=hw -filter_hw_device hw -i https://storage.rendi.dev/sample/big_buck_bunny_720p_stereo.avi -c:v h264_qsv output_gpu_qsv.mp4
```

More complicated, and less supported, encoding via [AMD GPUs via the Mesa VAAPI driver](https://trac.ffmpeg.org/wiki/Hardware/VAAPI)

## Misc

### FFmpeg Installation

List the formats your FFmpeg build supports:

```sh
ffmpeg -formats
```

List the codecs your FFmpeg build supports:

```sh
ffmpeg -codecs
```

### [FFprobe](https://ffmpeg.org/ffprobe.html)

It provides structured metadata about media files. Show detailed stream information of a video file:

```sh
ffprobe -show_streams -i https://storage.rendi.dev/sample/big_buck_bunny_720p_16sec.mp4
```

### Credits

- www.bigbuckbunny.org for the video and image files.
- Music credit to https://www.fiftysounds.com/music/neon-lights.mp3
- National Library of Congress and Paramount Pictures for Popeye https://www.loc.gov/item/2023602008/

## Possible future topics to add to the cheatsheet

- Frame rate configurations
- Elaborate about video bitrate settings
- Elaborate about audio bitrate settings
- Explanations of 1-pass and 2-pass encodings
- Overview of tune, presets, and profile options
- ASS subtitle format
- Explain more about creating web-optimized media
- Discuss online video streaming with FFmpeg
- Silence removal with FFmpeg
- Batch processing with FFmpeg

## Closing Remarks

If you have any questions, feedback or other commands you would like to see, write them in the Issues section.

I will be updating this document based on your feedback and requests.
