---
heroImage: '/android-scrcpy-screen-mirroring.svg'
title: 'Mastering Scrcpy: High-Performance Android Screen Mirroring'
description: 'Control your Android device from your PC with zero latency. Learn how to install and utilize Scrcpy for screen mirroring, recording, and keyboard integration.'
pubDate: 'May 18 2026'
---

There are dozens of apps that let you mirror your Android screen to a PC, but almost all of them suffer from severe latency, require you to install bloatware on your phone, or charge a premium subscription.

Enter **Scrcpy** (Screen Copy), a free, open-source command-line tool developed by Genymobile. It communicates directly over ADB, requires no app installation on your phone, and delivers high-resolution, incredibly low-latency screen mirroring (30~70ms).

Here is a guide to mastering this indispensable developer tool.

## Installation and Basic Usage

Scrcpy is available on Linux, Windows, and macOS.

On Linux (Ubuntu/Debian):
```bash
sudo apt install scrcpy
```

Ensure your Android device has **USB Debugging** enabled in Developer Options and is plugged into your PC.

To start mirroring, simply open your terminal and run:
```bash
scrcpy
```
Your phone's screen will instantly appear in a window on your desktop. You can use your mouse to tap and swipe, and your PC keyboard to type directly into Android apps.

## Advanced Scrcpy Commands

The power of Scrcpy lies in its command-line arguments.

### 1. Screen Recording
You can record your phone's screen natively through the ADB connection while mirroring.
```bash
# Record as an MP4 file
scrcpy --record file.mp4
```

### 2. Turn Off the Physical Screen
If you are controlling your phone entirely from your PC, leaving the physical phone screen on wastes battery and risks screen burn-in. You can mirror the display while turning off the physical screen.
```bash
scrcpy --turn-screen-off
# Or simply use the shortcut: scrcpy -S
```

### 3. Change Resolution and Bitrate
If you are connecting over a slow Wi-Fi network (Wireless ADB), you might want to lower the resolution and bitrate to maintain a smooth framerate.
```bash
# Limit resolution to 1024 on the longest side and bitrate to 4 Mbps
scrcpy --max-size 1024 --video-bit-rate 4M
```

### 4. Seamless Clipboard Sharing
By default, Scrcpy synchronizes the clipboards between your PC and your Android device.
*   Copy text on your PC (`Ctrl+C`), and simply paste it into a text field on your Android phone.
*   You can also drag and drop APK files directly into the Scrcpy window to install them, or drag other files to push them to the `/sdcard/Download/` folder.

### 5. Camera Mirroring (Use Android as a Webcam)
Scrcpy can directly mirror your phone's camera stream instead of the screen, effectively turning your phone into a high-quality webcam.
```bash
scrcpy --video-source=camera
```

### 6. Wireless Connection
Scrcpy works perfectly over Wi-Fi, provided you have enabled Wireless Debugging.

1. Connect to your phone over Wi-Fi using ADB: `adb connect <PHONE_IP>:5555`
2. Run Scrcpy as usual: `scrcpy`

If the wireless connection stutters, use the `--max-size` and `--video-bit-rate` flags mentioned above to optimize the stream.

Scrcpy is a masterclass in efficient software design. Whether you are presenting an app demo, replying to mobile messages without touching your phone, or recording tutorials, it is the ultimate tool for Android screen mirroring.
