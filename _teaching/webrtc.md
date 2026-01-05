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
