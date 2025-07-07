---
title: "音视频概述"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/video
date: 2025-07-07
---

# 音视频的处理流程及每个节点涉及到的协议

##  一、采集端 (Source Side)

### 视频/音频采集 (Video/Audio Acquisition)

    协议/标准：
        USB (Universal Serial Bus)： 连接摄像头、麦克风等外设。
        HDMI (High-Definition Multimedia Interface)： 高清视频输入接口。
        SDI (Serial Digital Interface)： 广播级数字视频接口。
        MIPI-CSI (Camera Serial Interface)： 移动设备中摄像头常用接口。
    模拟信号标准： 如 NTSC, PAL (但在数字化后就不再是模拟信号了)。

    数据格式：
        YUV, RGB： 原始视频像素数据格式。
        PCM (Pulse Code Modulation)： 原始音频采样数据格式。

### 预处理 (Pre-processing)

    协议/标准： 此环节主要涉及图像处理算法和 DSP（数字信号处理器）内部操作，不涉及外部协议。可能涉及内部的图像处理算法标准，如色彩空间转换算法。

### 编码 (Encoding)

    协议/标准 (编码器/Codec 标准)：

#### 视频编码标准：

        H.264 (MPEG-4 AVC)： 目前最广泛使用的视频编码标准。
        H.265 (HEVC - High Efficiency Video Coding)： H.264 的继任者，压缩效率更高。
        AV1 (AOMedia Video 1)： 开源、免版税的视频编码标准，旨在提供更高的压缩效率。
        VP9： 谷歌开发的视频编码标准，常用于 WebM。
        MPEG-2： 较早的视频编码标准，仍用于广播和 DVD。

#### 音频编码标准：

    AAC (Advanced Audio Coding)： 广泛使用，尤其在流媒体中。
    MP3 (MPEG-1 Audio Layer III)： 流行，但压缩率和音质不如 AAC。
    Opus： 开放、免版税的音频编码标准，低延迟，高音质，适用于实时通信。
    AC-3 (Dolby Digital)： 杜比数字音频标准。

    作用： 这些是定义如何压缩和解压缩数据的标准，而不是网络传输协议。

### 封装/复用 (Packaging/Multiplexing)

    协议/标准 (容器格式/Container Format)：
        MP4 (MPEG-4 Part 14)： 多媒体容器格式，可包含 H.264/H.265 视频、AAC 音频等。常用后缀 .mp4。
        FLV (Flash Video)： 早期 Adobe Flash Player 的流媒体格式。
        TS (MPEG Transport Stream)： 一种通用容器格式，将音视频等基本流复用到单一流中，适用于广播和流媒体（如 HLS）。常用后缀 .ts。
        WebM： 基于 Matroska 的开放、免版税格式，通常包含 VP8/VP9/AV1 视频和 Vorbis/Opus 音频。常用后缀 .webm。
        MOV (QuickTime File Format)： 苹果公司的多媒体容器格式。
        AVI (Audio Video Interleave)： 微软公司开发的较旧的容器格式。

## 二、传输 (Transmission)

    这是涉及网络传输协议的核心环节。

### 流媒体传输协议：

    RTMP (Real-Time Messaging Protocol)： 基于 TCP，由 Adobe 开发，早期直播（如 Flash 直播）的主流协议，特点是低延迟，但对防火墙不友好。
    HLS (HTTP Live Streaming)： 基于 HTTP，由苹果开发，将视频切片为小的 .ts 文件，并通过 .m3u8 播放列表索引。

#### 核心协议：HTTP/HTTPS (传输 .ts 片段和 .m3u8 索引)。

        文件格式：M3U8 (播放列表索引文件)，TS (视频切片)。
        MPEG-DASH (Dynamic Adaptive Streaming over HTTP)： 基于 HTTP，国际标准，与 HLS 类似，使用 .mpd (Media Presentation Description) 文件索引。

#### 核心协议：HTTP/HTTPS (传输视频片段和 .mpd 索引)。

    文件格式：MPD (媒体描述文件)，通常是分段 MP4 (fMP4)。
        WebRTC (Web Real-Time Communication)： 用于浏览器和移动应用之间的实时通信。
    主要传输协议：UDP (通常是 RTP/RTCP)，以实现低延迟。

#### 信令协议： 通常通过 HTTP/HTTPS (WebSocket) 或 SIP 等应用层协议进行会话建立和元数据交换。

#### 辅助协议：

    SDP (Session Description Protocol)： 用于描述会话（媒体类型、编解码器、传输协议等）。
    ICE (Interactive Connectivity Establishment)： 集合了 STUN 和 TURN，用于 NAT 穿越和防火墙打洞。
    STUN (Session Traversal Utilities for NAT)： 用于发现公网 IP 地址和端口。
    TURN (Traversal Using Relays around NAT)： 当 STUN 无法穿透 NAT 时，作为中继服务器转发数据。
    DTLS (Datagram Transport Layer Security)： 为 UDP 提供加密和安全。
    SRTP (Secure Real-time Transport Protocol)： RTP 的安全扩展，用于加密实时媒体流。
    RTSP (Real-Time Streaming Protocol) / RTP (Real-time Transport Protocol) / RTCP (Real-time Transport Control Protocol)： 常用于监控摄像头、IP 电话等。RTSP 控制会话，RTP 传输数据，RTCP 传输控制信息。通常基于 TCP 或 UDP。
    SRT (Secure Reliable Transport)： 开源，高弹性、低延迟的视频传输协议，用于高质量、低延迟的直播传输，尤其是在不稳定网络环境下。
    HLS/DASH over QUIC： 新兴趋势，利用 QUIC（基于 UDP 的多路复用传输协议）来优化 HLS/DASH 的传输性能。

#### 底层传输协议：

    TCP (Transmission Control Protocol)： 提供可靠的、面向连接的传输，适用于需要数据完整性的流（如 RTMP、HTTP）。
    UDP (User Datagram Protocol)： 提供无连接、不可靠但高效的传输，适用于对延迟敏感的实时流（如 RTP、WebRTC），通常在上层协议中实现自己的可靠性机制。

## 三、接收端/播放器 (Receiver/Player Side)

### 接收与缓存 (Reception & Buffering)

    协议： 客户端通过上述的流媒体传输协议 (如 HTTP, RTMP, WebRTC) 接收数据。此环节是基于接收到的数据进行内部处理，没有新增外部协议。

### 解封装/解复用 (Demultiplexing)

    协议/标准： 内部解析对应容器格式（MP4, TS, WebM 等）的协议规范，提取出编码后的视频流和音频流。

### 解码 (Decoding)

    协议/标准 (解码器/Codec 标准)： 对应编码阶段使用的视频编码标准（H.264, H.265, AV1, VP9 等）和音频编码标准（AAC, Opus, MP3 等）。解码器依据这些标准将压缩数据还原为原始像素和音频样本。

### 渲染与播放 (Rendering & Playback)

    协议/标准：

        图形 API： 如 DirectX (Windows), OpenGL (跨平台), Vulkan (跨平台), Metal (Apple) 用于将视频帧渲染到屏幕。
    音频 API： 如 WASAPI (Windows), Core Audio (macOS/iOS), OpenSL ES (Android) 用于将音频样本播放出来。
    时钟同步协议/算法： 内部算法，确保视频帧和音频样本按照正确的时间戳进行同步播放。