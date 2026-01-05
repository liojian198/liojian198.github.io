---
title: "webrtc"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/webrtc
date: 2026-01-05
---

# 概述

  WebRTC（Web Real-Time Communication）是一项允许网络浏览器和移动应用进行实时音视频通信和通用数据传输的开源技术。它最大的意义在于：不需要安装任何插件或第三方软件，直接在浏览器端就能实现点对点（P2P）的高性能通信。
  这个大框架的东西真多啊。

# 相关描述

## WebRTC 的核心价值

  超低延迟： 延迟通常在 500ms 以内（远低于 HLS 或 RTMP 直播）。

  免插件： 浏览器原生支持（Chrome, Safari, Firefox, Edge 等）。

  强大的丢包对抗： 针对不稳定的网络环境，内置了先进的拥塞控制和抗丢包算法。

  强制加密： 所有的音视频流都经过 SRTP 加密，安全性极高。

## WebRTC 的三大核心 API

  在开发层面，WebRTC 的功能主要由三个 JavaScript 接口组成：

  getUserMedia (采集)： 调用本地硬件（摄像头、麦克风），获取原始音视频流，并处理降噪、回声消除等。

  RTCPeerConnection (传输)： 这是最核心的部分。负责建立连接、处理丢包、加解密、带宽预估以及音视频的编码传输。

  RTCDataChannel (数据)： 除了音视频，还可以传输任何二进制数据（如游戏指令、文件、聊天信息），具有低延迟且可选可靠/不可靠传输模式。

## WebRTC 的建立连接流程（信令与穿透）

  虽然 WebRTC 是 P2P 的，但它无法直接感知对方。连接过程需要三类“服务器”辅助：

  信令服务器 (Signaling Server)： 像媒婆一样，帮双方交换“名片”（即 SDP，包含编码能力和媒体参数）。WebRTC 没定义具体的信令协议，通常开发者会用 WebSocket 或 HTTP 来实现。

  STUN 服务器： 用于“打洞”。帮设备获取自己的公网 IP 地址。

  TURN 服务器： 如果防火墙太严死活打不通洞，就得靠它进行流量中转。

## WebRTC 的底层协议栈

  WebRTC 并不是一个单一协议，而是多种协议的集合体：
  
  1. 层次协议作用媒体层RTP/RTCP负责音视频数据的实时传输和控制。
  2. 加密层DTLS/SRTP负责密钥协商和媒体数据加密。
  3. 连接层ICE/STUN/TURN负责穿越 NAT，建立网络通路。
  4. 传输层UDPWebRTC 优先使用 UDP 以保证实时性。

## WebRTC 线程模型

  在 WebRTC 内部，通常有三个核心线程在打配合：

  Signaling Thread (信令线程): 处理 API 调用和连接协商。

  Worker Thread (工作线程): 负责数据的采集、转换、分发。

  Network Thread (网络线程): 专门负责从 UDP 端口收发原始数据包。

  这种多线程设计确保了即使视频编码很耗资源，也不会导致网络包接收出现延迟。

## webRTC多工资流程及各个流程详解(P2P模式)

  WebRTC 的全流程是一个极其精密、多层联动的实时系统，旨在将延迟控制在 500ms 以内。
  这个流程可以拆解为：采集（Capture）、处理（Processing）、传输（Transport） 和 渲染（Playback） 四个核心阶段。

### 数据采集与预处理 (Capture & Pre-processing)

这是数据的源头，发生在使用者的本地设备上。

硬件调用： 浏览器或 Native 应用通过系统 API（如摄像头 V4L2/AVFoundation、麦克风 CoreAudio）采集原始数据。

音频预处理 (DSP)： * AEC (回声消除)： 防止对方的声音从你的扬声器出来后又进入你的麦克风。

ANS (降噪) & AGC (自动增益控制)： 去除环境杂音并保持音量稳定。

视频处理： 进行色彩空间转换（通常从 RGB 转为 YUV 420P）和缩放。

### 编码与打包 (Encoding & Packetization)

  原始数据量巨大，必须在发送前进行极高比例的压缩。

  视频编码： 使用 H.264、VP8/VP9 或 AV1。

  Simulcast (联播)： 编码器可能同时产生多个分辨率（如 180p, 360p, 720p），以便服务器根据不同订阅者的网速动态下发。

  音频编码： 默认使用 Opus，它能根据带宽自动在窄带语音和全带音乐间切换。

  RTP 打包： 编码后的数据被切分成一个个符合 RTP 协议 的数据包，每个包都被打上序列号和时间戳。


### 连接建立与网络传输 (Signaling & Transport)

  这是 WebRTC 最复杂的部分，解决了“怎么在复杂的互联网找到对方”的问题。

  信令 (Signaling)： 双方交换 SDP (包含媒体能力) 和 ICE Candidates (包含网络地址)。

  安全加固 (DTLS)： 在传输前进行密钥交换，确保所有 RTP 数据都被加密为 SRTP。

  UDP 传输： 数据优先走 UDP 以降低延迟。

  QoS (服务质量保证)：

    NACK (重传)： 发现丢包时请求对方重推。

    FEC (前向纠错)： 随包发送冗余信息，丢包后可自行恢复。

    BWE (带宽估算)： 实时探测网速，动态调整编码码率。

### 接收与播放 (Decoding & Rendering)

  数据包到达对方设备后，需要经过最后的平滑处理。

  Jitter Buffer (抖动缓冲区)： 这是消除卡顿的关键。

  数据包在网络中可能乱序或延时到达。缓冲区会将包重新排序、去重，并等待迟到的包，确保输出给解码器的数据是连续且有序的。

  音频 NetEq 处理： 专门处理音频抖动。如果包丢了，它会利用算法“伪造”一小段极其相似的声音（PLC 丢包补偿），让你听不出停顿。

  解码与音画同步： 视频解码器还原画面。通过 RTCP SR (Sender Report) 记录的绝对时间映射，确保声音和嘴型完全同步。

  渲染： 最终通过显卡渲染到 <video> 标签或系统窗口，通过声卡播放声音。


### 总结流程图

|步骤|角色|核心技术/协议|
|--|--|--|
|1| 采集|硬件接口,MediaDevices / DSP (AEC/ANS)|
|2| 压缩|编码器,H.264 / VP9 / Opus|
|3| 握手|信令通道,SDP / ICE / STUN|
|4|传输|加密通道|SRTP / DTLS / UDP|
|5| 抗抖动|接收端缓冲,Jitter Buffer / NetEq|
|6| |呈现|渲染引擎|GPU Rendering / Audio Renderer|
