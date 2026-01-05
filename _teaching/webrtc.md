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

## webRTC多工作流程及各个流程详解(P2P模式)

  WebRTC 的全流程是一个极其精密、多层联动的实时系统，旨在将延迟控制在 500ms 以内。
  这个流程可以拆解为：采集（Capture）、处理（Processing）、传输（Transport） 和 渲染（Playback） 四个核心阶段。

### 数据采集与预处理 (Capture & Pre-processing)

  这是数据的源头，发生在使用者的本地设备上。

    硬件调用： 浏览器或 Native 应用通过系统 API（如摄像头 V4L2/AVFoundation、麦克风 CoreAudio）采集原始数据。

    音频预处理 (DSP)： * AEC (回声消除)： 防止对方的声音从你的扬声器出来后又进入你的麦克风。

    ANS (降噪) & AGC (自动增益控制)： 去除环境杂音并保持音量稳定。

    视频处理： 进行色彩空间转换（通常从 RGB 转为 YUV 420P）和缩放。

  在视频进入编码器之前，有一系列隐藏的处理步骤：

    采集线程 (Capturer Thread)： 并不只是拉取图像，它会根据 CPU 负载动态调整采集的分辨率和帧率。
  
    图像增强： 包含人脸检测、亮度自适应、甚至去噪算法，确保进入编码器的画面是高质量的。

    颜色转换： 硬件采集通常是 RGB，但编码器需要 YUV420 格式，这个转换过程通常在 GPU 中通过 Shader 完成以减少延迟。

### 编码与打包 (Encoding & Packetization)

  原始数据量巨大，必须在发送前进行极高比例的压缩。

    视频编码： 使用 H.264、VP8/VP9 或 AV1。

    Simulcast (联播)： 编码器可能同时产生多个分辨率（如 180p, 360p, 720p），以便服务器根据不同订阅者的网速动态下发。

    音频编码： 默认使用 Opus，它能根据带宽自动在窄带语音和全带音乐间切换。

    RTP 打包： 编码后的数据被切分成一个个符合 RTP 协议 的数据包，每个包都被打上序列号和时间戳。

    编码器输出的不是一个完整的文件，而是一帧帧数据。

  RTP 扩展头 (Header Extensions)： WebRTC 会在 RTP 包头塞进很多“额外信息”，比如：

    Abs-send-time: 记录包发出的绝对时间（精确到毫秒），用于接收端计算网络抖动。

    Transport-wide Sequence Number: 给所有的包（不论是音还是视）统一打号，用于全局带宽估算（GCC 算法）。

  FEC 与 RTX 冗余：

    FEC (前向纠错): 编码器会根据当前的丢包率，额外产生一些“冗余包”。如果中间丢了一个，接收端可以通过余下的包直接算出来。

    RTX (重传): 丢包太严重时，会把丢掉的包重新塞进一个专门的“重传流”里。


### 连接建立与网络传输 (Signaling & Transport)

  这是 WebRTC 最复杂的部分，解决了“怎么在复杂的互联网找到对方”的问题。

  信令 (Signaling)： 双方交换 SDP (包含媒体能力) 和 ICE Candidates (包含网络地址)。

  安全加固 (DTLS)： 在传输前进行密钥交换，确保所有 RTP 数据都被加密为 SRTP。

  UDP 传输： 数据优先走 UDP 以降低延迟。

  QoS (服务质量保证)：

    NACK (重传)： 发现丢包时请求对方重推。

    FEC (前向纠错)： 随包发送冗余信息，丢包后可自行恢复。

    BWE (带宽估算)： 实时探测网速，动态调整编码码率。

  这是 WebRTC 的大脑。它实时监测网络带宽，防止链路拥塞：

    延迟探测 (Delay-based Controller): 如果包到达的时间间隔越来越长，说明网络节点开始排队了，系统会立刻下调码率。

    丢包探测 (Loss-based Controller): 如果出现丢包，说明带宽已经到极限了。

    联动： 这种估算会直接反馈给 编码器。如果你网速变差，你会发现画面瞬间变模糊（码率降低），但这保证了通话不会中断。

### 接收与播放 (Decoding & Rendering)

  数据包到达对方设备后，需要经过最后的平滑处理。

    Jitter Buffer (抖动缓冲区)： 这是消除卡顿的关键。

    数据包在网络中可能乱序或延时到达。缓冲区会将包重新排序、去重，并等待迟到的包，确保输出给解码器的数据是连续且有序的。

    音频 NetEq 处理： 专门处理音频抖动。如果包丢了，它会利用算法“伪造”一小段极其相似的声音（PLC 丢包补偿），让你听不出停顿。

    解码与音画同步： 视频解码器还原画面。通过 RTCP SR (Sender Report) 记录的绝对时间映射，确保声音和嘴型完全同步。

    渲染： 最终通过显卡渲染到 <video> 标签或系统窗口，通过声卡播放声音。

  数据包到达后，面临的是“乱序”和“抖动”。

    NetEq (音频专用): 这是 Google 的黑科技。它不仅是一个缓冲区，还会做：

    Accelerate (加速播放): 如果堆积的包太多，它会轻微加快语速（你听不出来），把缓冲区清空。

    Preemptive Expand (减速播放): 如果包还没到，它会拉长上一段声音的尾音。

    Jitter Buffer (视频专用): 它会等待一帧的所有 RTP 包集齐。如果某个包迟到了，它会卡住一会儿，通过算法决定是“等重传”还是“直接跳过这一帧”。

### 渲染与同步：最后的对齐

音画同步 (Lip-Sync): 接收端通过 RTCP SR (Sender Report) 将音频的 RTP 时间轴和视频的 RTP 时间轴映射到同一个 NTP 时间（墙上时钟）上。

垂直同步 (VSync): 最终视频帧投递给显卡时，会配合显示器的刷新率，防止画面撕裂。


### 总结流程图

|步骤|角色|核心技术/协议|
|--|--|--|
|1| 采集|硬件接口,MediaDevices / DSP (AEC/ANS)|
|2| 压缩|编码器,H.264 / VP9 / Opus|
|3| 握手|信令通道,SDP / ICE / STUN|
|4| 传输|加密通道|SRTP / DTLS / UDP|
|5| 抗抖动|接收端缓冲|Jitter Buffer / NetEq|
|6| 呈现|渲染引擎|GPU Rendering/Audio Renderer|

## webRTC多工作流程及各个流程详解(SFU模式）

  引入 LiveKit 服务器后，WebRTC 的四个阶段从传统的“点对点 (P2P)” 演变成了 “端-服务器-端 (Pub/Sub)” 的 SFU（Selective Forwarding Unit，选择性转发单元）架构。
  在这种架构下，LiveKit 充当了“智能路由”的角色。以下是加入 LiveKit 后各阶段的详细处理细节：

### 采集与预处理 (Capture & Ingress)

  在 LiveKit 体系中，采集不仅限于浏览器，还包括外部流的拉取。

    客户端采集： SDK（JS/Swift/Kotlin）调用本地硬件，进行 3A 算法（回声消除 AEC、降噪 ANS、增益 AGC）预处理。
    Ingress（外部输入）： * WHIP/RTMP 接入： 如果你使用 OBS 推流，LiveKit Ingress 节点会接收流并将其转码。
    GStreamer 管道： Ingress 内部使用 GStreamer 将非 WebRTC 格式（如 RTMP）拆解，转换为符合 LiveKit 规范的 RTP 裸流。
    Simulcast（联播）准备： 采集端会根据配置，同时编码出高、中、低三档分辨率（例如 1080p, 720p, 360p），为服务器的后续分发做准备。

### 处理与逻辑层 (Processing & SFU Logic)

  这是 LiveKit 服务器的核心，它不只是搬运工，更是“指挥官”。

  SFU 转发逻辑： LiveKit Server 不会对媒体流进行重编码（除非是 Ingress），它通过 RTP 头部修改 快速转发包，延迟仅增加几毫秒。
  丢包补偿 (NACK/RTX)： 服务器维护一个滑动窗口缓冲区。当订阅者反馈丢包（NACK）时，LiveKit 直接从自己的内存缓冲区中重传该包，而不需要请求原始发布者，极大缩短了修复时间。
  带宽预估 (BWE)： 服务器实时监测每个订阅者的下行带宽。如果观众网速变慢，LiveKit 会自动切换 Simulcast 图层（例如从 720p 切到 360p），确保视频不卡顿。
  序列号重写 (SN Rewriting)： 当服务器在不同图层间切换时，会实时修改 RTP 包的序列号和时间戳，使客户端解码器感知不到流的切换。

### 传输阶段 (Transport & Signaling)

加入服务器后，传输变得更加稳健和可控。

信令交互 (WebSocket)： 所有的连接协商（SDP Offer/Answer）都通过 WebSocket 走专门的信令通道，而非 P2P 的私有协议。
中转 (TURN/UDP/TCP)： * LiveKit 内置了 TURN 服务。如果用户在严格的企业内网，它会自动切换到 TLS 443 端口 封装 UDP 包进行传输，确保连接成功率接近 100%。
分发网格 (Distributed Mesh)： 在大规模场景下，LiveKit 节点之间通过 Redis 协调。如果房间在 A 节点，而观众在 B 节点，A 会将媒体流通过内网高带宽链路转发给 B。


### 渲染与播放 (Playback & Egress)

最终画面在用户终端或录制服务中呈现。

Jitter Buffer（客户端）： LiveKit SDK 接管了浏览器的缓冲区管理，根据服务器下发的网络状况动态调整缓冲深度。
Egress（外部输出）：
合成录制： 如果开启录制，LiveKit Egress 会启动一个 Headless Chrome，像真实用户一样订阅房间里的所有音视频轨道，渲染成画面后，再用 FFmpeg 压制成 MP4 或推向 Youtube。
AI 代理接入 (Agents)： 如果房间里有 AI，AI 进程会作为特殊的“参与者”订阅音频轨道，通过 VAD (语音活跃检测) 提取有效片段，送入 STT/LLM 管道进行处理。

### LiveKit 模式下的关键参数对比

|阶段|传统 WebRTC (P2P)|LiveKit (SFU) 模式|
|带宽消耗|随参与者增加指数增长|恒定（每个发布者仅推一份流）|
|重传机制|终端对终端|服务器代理重传 (快得多)|
|分辨率切换|几乎没有|动态 Simulcast 自动切换|
|防火墙穿透|依赖第三方 STUN/TURN|内置 TLS 隧道支持|
