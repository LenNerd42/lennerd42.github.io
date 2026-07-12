---
layout: post
title: Using MP4 with DaVinci Resolve on Linux
description: A quick and comprehensive guide for getting MP4 files working in DaVinci Resolve on Linux.
tags: [davinci resolve, mp4, linux]
categories: [Video Editing]
date: 2026-07-12 15:20 +0200
media_subpath: /assets/img/posts/davinci-mp4-linux/
---

DaVinci Resolve is a great free video editor available on all major platforms, including Linux. However, the free Linux version suffers from one critical flaw: **there is no built-in support for common MP4 codecs like H.264/H.265** (due to [licensing issues](https://jamesnorth.net/post/mp4-with-resolve-on-gnu-linux)).\\
This really took me by surprise the first time I used it after switching from Windows. If you've run into this limitation, fear not! The guide shows how to get around this limitation and use MP4 files with DaVinci Resolve on Linux.

> If you're on an NVIDIA RTX 30 Series card or newer, skip to ["A Note on AV1 for RTX 30+ Series Users"](#a-note-on-av1-for-rtx-30-series-users), as there's are better options available for these cards.
{: .prompt-info}

## Just Give me the Details!

Okay, I won't bore you with any more talk. Here is an overview of the process I use for recording, importing and exporting video files with DaVinci.

![Light mode only](workflow-light.png){: .light }
![Dark mode only](workflow-dark.png){: .dark }
_An overview of my entire process_

The entire process only requires OBS Studio, `ffmpeg` and DaVinci Resolve, of course. OBS Studio can be installed from [Flathub](https://flathub.org/en/apps/com.obsproject.Studio) and `ffmpeg` can be installed using `sudo apt install ffmpeg` or `sudo dnf install ffmpeg` depending on your distro.

### Recording

For recording, I use OBS Studio with the recording settings shown below. This results in an MP4 file, using H.264 as the video codec and AAC as the audio codec. This is ideal for playback in VLC and sharing the video, i.e. on platforms like Discord or YouTube.

![OBS Studio recording settings: Hybrid MP4 (.mp4), x264 video codec, libfdk AAC audio codec](obs-settings.png)

### Conversion for Input Media

The free version of DaVinci on Linux doesn't support either H.264 or AAC, so we need to convert videos to VP9 (video codec) and MP3 (audio codec).
This can be done with the command below. Make sure to replace the input and output filenames with whatever you see fit:\\
`ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libmp3lame -b:a 320k output.mp4`

This produces an VP9 + MP3 encoded file that can be imported into DaVinci.

### DaVinci Exports

DaVinci can export to a variety of formats, but I've found that using the DNxHR codec with the LB (low bandwidth) setting strikes a nice balance between quality and file size. Below are my export settings for both video and audio. Feel free to tweak these settings to fit your own needs.

![DaVinci Resolve video export settings: Format QuickTime, Codec Avid DNxHR, Type Avid DNxHR LB 12-bit](davinci-export-video.png)

![DaVinci Resolve audio export settings: Codec MP3, Bitrate 320 Kb/s](davinci-export-audio.png)

### Conversion of Output Media

To convert DaVinci's output from a MOV to a H.264 + AAC encoded MP4 again, you can use this command if you're using an NVIDIA GPU. Again, adjust file names as you see fit:

`ffmpeg -i input.mov -c:v h264_nvenc -preset p6 -cq 23 -pix_fmt yuv420p -c:a aac -b:a 320k output.mp4`

AMD users need to use `-c:v h264_amf` instead, as `h264_nvenc` refers to the H.264 NVIDIA GPU encoder (which obviously isn't available for AMD users):

`ffmpeg -i input.mov -c:v h264_amf -preset p6 -cq 23 -pix_fmt yuv420p -c:a aac -b:a 320k output.mp4`

## A Note on AV1 for RTX 30+ Series Users

For owners of an NVIDIA RTX 40 Series or 50 Series, there's good news: you can encode your MP4 files using AV1 (available in OBS as the "AOM AV1" video encoder). DaVinci Resolve can both import and export AV1 encoded MP4 files.

If you're on an RTX 30 Series card, you can still import AV1 encoded media into DaVinci, but you won't be able to export with this codec, as the 30 Series cards lack hardware encoding support for AV1. You're gonna have to use my method described in [the section above](#conversion-of-output-media) for exporting media.

## My DaVinci Transcoder Utility - a way to automate the process

Since the process described above relies on complicated-looking `ffmpeg` commands and is tedious to repeat for several video files, I've created a small utility application in Python. It's mostly meant for internal use, but if you want, you can pull the code from GitHub [here](https://github.com/LenNerd42/davinci-transcoder-utility).

If there's any interest, I might offer pre-built releases in the future, but for now you'll have to build it yourself by following the instructions in the README. Oh, and it **won't work for AMD users** out of the box, so you'll have to modify the code yourself. Have fun!

## Further Reading

[DaVinci Transcoder Utility](https://github.com/LenNerd42/davinci-transcoder-utility)

[Using DaVinci Resolve on Linux With MP4—Questions, Answers, and Tips](https://jamesnorth.net/post/mp4-with-resolve-on-gnu-linux)

[DaVinci Resolve 20: Supported Formats and Codecs](https://documents.blackmagicdesign.com/SupportNotes/DaVinci_Resolve_20_Supported_Codec_List.pdf?_v=1723705210000)
