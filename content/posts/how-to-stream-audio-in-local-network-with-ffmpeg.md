---
title: "How to stream audio locally with FFMpeg?"
date: 2024-10-31
series: ["ffmpeg", "vlc"]
weight: 1
tags: ["ffmpeg", "vlc"]
author: "Arto Salminen"
---

Streaming audio locally can be useful for broadcasting live audio to multiple devices or applications within the same network. With FFmpeg, this setup becomes simple, allowing you to stream audio directly from your microphone to any player that supports UDP streaming, like VLC. This guide explains how to set up FFmpeg on Windows, configure a low-latency stream, and tune it for optimal performance.

## Step 1: Install FFmpeg on Windows

To use FFmpeg on Windows, you’ll need to download and install it. Follow these steps:

1. **Download FFmpeg**: Go to the official [FFmpeg website](https://ffmpeg.org/download.html) and download the "FFmpeg Release" for Windows. This will typically download as a `.zip` file.
2. **Extract the Files**: Once downloaded, unzip the FFmpeg folder to a location of your choice (e.g., `C:\ffmpeg`).
3. **Add FFmpeg to the System Path**:
   - Open the **Start Menu** and search for “Environment Variables.”
   - Select **Edit the system environment variables**.
   - In the **System Properties** window, click **Environment Variables**.
   - Under **System Variables**, find the `Path` variable, select it, and click **Edit**.
   - Add the path to the `bin` folder of FFmpeg (e.g., `C:\ffmpeg\bin`).
   - Click **OK** to save and close each window.
4. **Verify Installation**:
   - Open **Command Prompt** and type `ffmpeg -version`.
   - If installed correctly, FFmpeg should display its version and configuration information.

## Step 2: Streaming Audio from Microphone Locally

Now that FFmpeg is installed, you can start streaming audio from your microphone. Here’s a command to stream audio with low latency, optimized for UDP on the local network.

```bash
ffmpeg -f dshow -i audio="Microphone (AUDIO 2.0)" -acodec libmp3lame -b:a 128k -bufsize 32k -fflags nobuffer -flags low_delay -flush_packets 1 -max_delay 0 -f mp3 udp://127.0.0.1:1234
```

### Command Breakdown

- **`-f dshow`**: Specifies DirectShow as the input format on Windows, which allows FFmpeg to access your microphone.
- **`-i audio="Microphone (AUDIO 2.0)"`**: Selects the microphone device. Replace `"Microphone (AUDIO 2.0)"` with the exact name of your microphone as recognized by FFmpeg.
- **`-acodec libmp3lame`**: Sets the MP3 codec for encoding audio, providing good compression with manageable latency.
- **`-b:a 128k`**: Sets the audio bitrate to 128 kbps, balancing quality and bandwidth usage.
- **`-bufsize 32k`**: Sets a small buffer size, reducing latency by speeding up the buffer fill time.
- **`-fflags nobuffer`**: Reduces buffering on the input side to decrease overall latency.
- **`-flags low_delay`**: Enables low-latency mode for faster processing.
- **`-flush_packets 1`**: Flushes packets immediately after encoding, reducing delay.
- **`-max_delay 0`**: Sets the maximum delay to zero, minimizing additional buffering.
- **`-f mp3`**: Specifies the MP3 format for the output stream.
- **`udp://127.0.0.1:1234`**: Sends the audio stream to UDP port `1234` on the local machine. Change `127.0.0.1` to your computer’s IP if you want other devices on the network to access the stream.

## Step 3: Accessing the Stream in VLC Media Player

To listen to the streamed audio, use a media player that supports UDP streaming, such as VLC:

1. **Open VLC**: Launch VLC on the same computer or any other device within the local network.
2. **Open Network Stream**:
   - In VLC, go to **Media > Open Network Stream**.
   - Enter the following URL: `udp://@:1234`.
3. **Adjust Buffering (if necessary)**:
   - VLC might buffer the stream by default, which can add a delay. To reduce it, go to **Tools > Preferences > Input/Codecs**.
   - Set the **Network caching** value to a low number (e.g., `300 ms`).

## Fine-Tuning for Optimal Performance

Depending on your network and system performance, you may want to experiment with these additional tips:

- **Adjust Bitrate**: Lowering the bitrate with `-b:a 96k` or `-b:a 64k` will use less bandwidth, reducing the load on the network, which can help with latency.
- **Experiment with Audio Format**: While MP3 is a good balance between quality and latency, other formats like AAC may offer lower latency depending on your setup. Try replacing `libmp3lame` with `aac` for comparison.
- **Use a Different Port**: If port `1234` is already in use, you can replace it with an open port number.

## Wrapping Up

By following these steps, you’ve set up a low-latency audio stream on your local network using FFmpeg. This command is a great starting point for live audio applications such as broadcasting, intercom setups, or audio monitoring in a local environment. For those who need higher performance, additional tuning options with FFmpeg and VLC settings can help achieve an ideal balance between quality and latency. 

Happy streaming!
