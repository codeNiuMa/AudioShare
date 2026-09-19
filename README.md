# AudioShare - 将 Windows 音频实时分享到 Android 设备

[![最新版本](https://img.shields.io/github/v/release/codeNiuMa/AudioShare.svg?label=latest)](https://github.com/codeNiuMa/AudioShare/releases/latest)

[English](./README.en.md) | 简体中文

AudioShare 可以将 Windows 电脑正在播放的声音实时传输到一台或多台 Android 设备。它支持 USB 和 Wi-Fi 两种连接方式，并可为不同设备选择立体声、左声道或右声道。

本分支专注于电脑音频共享，不包含云音乐、网页远程管理、本地音乐播放器、歌词、多机云音乐同步、斐讯 R1 氛围灯和 Android 开机自启功能。

## 功能特色

- 实时 PCM 音频传输，兼顾低延迟和播放稳定性。
- 支持多台 Android 设备同时连接。
- 每台设备可独立选择立体声、左声道、右声道或禁用。
- 支持 USB（ADB 转发）和局域网 Wi-Fi（TCP）连接。
- Wi-Fi 模式支持局域网设备自动发现，也可以手动输入地址。
- Windows 音量可同步到 Android 设备。
- Android 使用前台服务持续接收音频，播放期间使用 WakeLock 避免休眠中断。

## 工作原理

```text
Windows WASAPI/NAudio
        ↓
有界 PCM 发送队列
        ↓
TCP 或 ADB LocalSocket 转发
        ↓
Android TcpService
        ↓
AudioTrack 实时播放
```

Wi-Fi 模式下，Android 会定期发送 UDP 广播，Windows 在可用的发现端口上接收设备地址。音频数据和心跳数据通过同一条 TCP 连接串行发送，避免数据帧互相穿插。

## v1.7.14 稳定性优化

- 删除已经失效的氛围灯时间同步命令。
- Windows 音频发送改为最多缓存 4 个 PCM 包的有界队列。
- 短暂网络抖动时不再立即丢弃新音频；积压过多时丢弃最旧数据，以避免延迟持续增加。
- 音频和心跳写入串行化，避免 TCP 数据帧交错。
- 重连时旧发送队列不会继续向新连接写入数据。
- 修复 Windows UDP 自动发现过程中多余 Socket 未释放的问题。
- Android 服务销毁时会释放广播定时器、Socket、AudioTrack、输出流和 WakeLock。
- Activity 退出后会解除服务监听器，降低界面和服务之间的生命周期泄漏风险。

## 使用方法

### 下载文件

从本仓库的 [最新 Release](https://github.com/codeNiuMa/AudioShare/releases/latest) 下载：

- `AudioShare.exe`：常规 Windows 版本。
- `AudioShare.net6.exe`：.NET 6 单文件版本，需要安装 .NET 6 Desktop Runtime x64。
- `AudioShare.apk`：Android 客户端。
- `adb.exe`、`AdbWinApi.dll`、`AdbWinUsbApi.dll`：USB 连接和自动安装所需文件；仅使用 Wi-Fi 或电脑已经配置 ADB 时可以不下载。

建议将 EXE、APK 和 ADB 相关文件放在同一个文件夹中。使用 `AudioShare.net6.exe` 且需要自动安装 APK 时，请手动安装 `AudioShare.apk`，或将其复制并重命名为 `AudioShare.net6.apk`。

### USB 连接

1. 在 Android 设备上启用开发者选项和 USB 调试。
2. 使用 USB 数据线连接电脑，并在 Android 设备上允许这台电脑进行 USB 调试。
3. 将 `AudioShare.exe`、`AudioShare.apk` 和三个 ADB 文件放在同一文件夹。
4. 启动 Windows 客户端，在“USB 设备”列表中选择设备和声道。
5. 点击连接。Android 应用缺失或版本不一致时，Windows 会尝试自动安装匹配版本。

![Windows USB 界面](./images/windows.png)

### Wi-Fi 连接

1. 在 Android 设备上安装并打开 `AudioShare.apk`。
2. 确保电脑和 Android 设备位于同一局域网，且路由器没有开启客户端隔离。
3. 启动 Windows 客户端，等待其自动发现 Android 设备。
4. 如果没有自动发现，可在 Android 界面查看地址和端口，并在 Windows 中手动输入，例如 `192.168.1.20:8088`。
5. 选择声道并连接。首次运行时请允许 Windows 防火墙中的局域网访问。

![Windows Wi-Fi 界面](./images/windows-wifi.png)

### Android 端

Android 应用启动后会显示当前可用的 TCP 地址。连接期间应保留 AudioShare 的前台服务通知；它用于保证应用可以在后台持续接收音频。

本版本不包含开机自启。Android 设备重启后，需要重新打开一次 AudioShare。

![Android 界面](./images/android.png)

## 常见问题

### USB 列表中没有设备

- 检查 USB 调试是否已开启。
- 重新插拔数据线，并确认 Android 上已经接受调试授权。
- 确认 ADB 三个文件与 EXE 位于同一文件夹。
- 关闭其他正在占用 ADB 的工具后重试。

### Wi-Fi 无法自动发现

- 确认两台设备位于同一局域网，并关闭访客网络或 AP 隔离。
- 允许 AudioShare 通过 Windows 防火墙的专用网络。
- 直接使用 Android 界面显示的 `IP:端口` 手动连接。

### 偶尔出现卡顿

- 优先使用 5 GHz Wi-Fi、有线转 Wi-Fi 接入点或 USB 连接。
- 避免电脑或手机同时进行大流量下载。
- 先使用 48 kHz 或 44.1 kHz；较高采样率会明显增加网络吞吐量。
- 多设备播放时，网络负载会随设备数量增加。

## 自动构建和发布

仓库中的 [GitHub Actions 工作流](./.github/workflows/release.yml) 会在推送 `v*.*.*` 标签时构建并发布：

- `AudioShare.exe`
- `AudioShare.net6.exe`
- `AudioShare.apk`
- ADB 运行文件

最新安装文件请从 [Releases](https://github.com/codeNiuMa/AudioShare/releases) 下载。

## 项目来源

本项目基于 [HeHang0/AudioShare](https://github.com/HeHang0/AudioShare) fork，并针对纯 Windows 音频共享场景进行了功能精简和稳定性优化。
