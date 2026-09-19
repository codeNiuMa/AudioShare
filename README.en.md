# AudioShare - Stream Windows audio to Android devices

[![Latest release](https://img.shields.io/github/v/release/codeNiuMa/AudioShare.svg?label=latest)](https://github.com/codeNiuMa/AudioShare/releases/latest)

English | [简体中文](./README.md)

AudioShare streams the audio currently playing on a Windows computer to one or more Android devices. It supports USB and Wi-Fi connections, with independent stereo, left-channel, or right-channel output for each device.

This fork focuses on computer audio sharing. It does not include cloud music services, web-based remote management, a local music player, lyrics, cloud-music multi-device synchronization, Phicomm R1 lighting, or Android boot auto-start.

## Features

- Real-time PCM audio streaming with low latency and stable playback.
- Multiple Android devices can be connected at the same time.
- Each device can independently use stereo, left, right, or disabled output.
- USB through ADB forwarding and LAN Wi-Fi through TCP are supported.
- Wi-Fi devices can be discovered automatically or entered manually.
- Windows volume can be synchronized to Android devices.
- Android uses a foreground service and a playback WakeLock to keep streaming in the background.

## Architecture

```text
Windows WASAPI/NAudio
        ↓
Bounded PCM send queue
        ↓
TCP or ADB LocalSocket forwarding
        ↓
Android TcpService
        ↓
Real-time AudioTrack playback
```

In Wi-Fi mode, Android periodically sends UDP broadcasts and Windows listens on the first available discovery port. Audio and heartbeat data are serialized over the same TCP connection so frames cannot interleave.

## v1.7.14 stability improvements

- Removed the obsolete lighting synchronization command.
- Added a bounded Windows audio queue that holds up to four PCM packets.
- Short network stalls no longer immediately discard new audio. When the queue is full, the oldest packet is dropped to prevent continuously increasing latency.
- Serialized audio and heartbeat writes to prevent TCP frame corruption.
- Prevented an old send queue from writing data into a reconnected client.
- Fixed unused UDP discovery sockets on Windows.
- Android now releases its broadcast timer, sockets, AudioTrack, output stream, and WakeLock when the service is destroyed.
- The Activity unregisters its service listener when closed, reducing lifecycle leak risks.

## Usage

### Download files

Download the following files from this repository's [latest release](https://github.com/codeNiuMa/AudioShare/releases/latest):

- `AudioShare.exe`: standard Windows build.
- `AudioShare.net6.exe`: .NET 6 single-file build; requires the x64 .NET 6 Desktop Runtime.
- `AudioShare.apk`: Android client.
- `adb.exe`, `AdbWinApi.dll`, and `AdbWinUsbApi.dll`: required for USB connections and automatic APK installation. They are optional when only Wi-Fi is used or ADB is already installed.

Keep the EXE, APK, and ADB files in the same folder. When using `AudioShare.net6.exe` with automatic APK installation, install `AudioShare.apk` manually or copy it as `AudioShare.net6.apk`.

### USB connection

1. Enable Developer options and USB debugging on the Android device.
2. Connect the device with a USB cable and approve the computer's debugging authorization on Android.
3. Put `AudioShare.exe`, `AudioShare.apk`, and the three ADB files in the same folder.
4. Start the Windows client and select the device and audio channel from the USB device list.
5. Connect. If the Android app is missing or its version differs, Windows will try to install the matching APK automatically.

![Windows USB interface](./images/windows.png)

### Wi-Fi connection

1. Install and open `AudioShare.apk` on the Android device.
2. Connect Windows and Android to the same LAN and make sure client isolation is disabled on the router.
3. Start the Windows client and wait for it to discover the Android device.
4. If discovery does not work, enter the address shown by Android manually, for example `192.168.1.20:8088`.
5. Select an audio channel and connect. Allow private-network access in Windows Firewall when prompted.

![Windows Wi-Fi interface](./images/windows-wifi.png)

### Android

The Android app displays its available TCP addresses after startup. Keep the AudioShare foreground-service notification enabled while connected; it allows the app to continue receiving audio in the background.

This version does not start automatically after an Android reboot. Open AudioShare once after restarting the device.

![Android interface](./images/android.png)

## Troubleshooting

### No device appears in the USB list

- Make sure USB debugging is enabled.
- Reconnect the cable and accept the debugging authorization prompt on Android.
- Verify that the three ADB files are in the same folder as the EXE.
- Close other tools that may be using ADB and try again.

### Wi-Fi discovery does not find the device

- Confirm that both devices are on the same LAN and disable guest-network or AP isolation.
- Allow AudioShare through Windows Firewall on private networks.
- Connect manually using the `IP:port` displayed by Android.

### Occasional audio stutter

- Prefer 5 GHz Wi-Fi, a wired access point, or USB.
- Avoid large simultaneous downloads on the computer or phone.
- Start with 48 kHz or 44.1 kHz; higher sample rates require significantly more network bandwidth.
- Network load increases with every connected playback device.

## Automated builds and releases

The repository's [GitHub Actions workflow](./.github/workflows/release.yml) builds and publishes the following files when a `v*.*.*` tag is pushed:

- `AudioShare.exe`
- `AudioShare.net6.exe`
- `AudioShare.apk`
- ADB runtime files

Download current builds from [Releases](https://github.com/codeNiuMa/AudioShare/releases).

## Origin

This project is forked from [HeHang0/AudioShare](https://github.com/HeHang0/AudioShare) and has been streamlined and stabilized for Windows-to-Android audio sharing.
