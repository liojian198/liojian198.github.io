---
title: "livekit 简介"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/livekit
date: 2026-01-05 15:58:00
---

# 概述

  LiveKit 是目前全球最受欢迎的开源实时音视频 (RTC) 框架之一。它的核心目标是让开发者能够像搭建 Web 页面一样简单地构建高质量、低延迟的音视频通话、直播以及 AI 实时语音交互系统。
  如果说传统的 WebRTC 是原生的“手工作业”，那么 LiveKit 就是一套工业级的“自动化生产线”。

  https://github.com/livekit/livekit
  https://docs.livekit.io/agents/

# 简介

## LiveKit 的核心组件

  LiveKit 并不是单一的软件，而是一个完整的生态系统：
    LiveKit Server (SFU): 核心引擎。采用 Go 语言编写，负责音视频流的选择性转发、房间状态管理和集群调度。它不进行编解码，因此性能极高。
    Client SDKs: 覆盖了几乎所有主流平台（Web, iOS, Android, Flutter, Unity, React Native, Go, Python, ESP32）。
    LiveKit Agents: 专门为 AI 实时对话 设计的框架。让 AI（如 GPT-4, DeepSeek）能以极低延迟（<1s）接入音视频房间。
    Egress (出口服务): 负责将房间内容录制成 MP4，或转码推流到 YouTube/Bilibili (RTMP)。
    Ingress (入口服务): 允许外部流（如 OBS 的 RTMP 或 WHIP）进入 LiveKit 房间。

## 为什么选择 LiveKit？（四大优势）

### A. 极致的开发者体验

  在原生 WebRTC 中，你需要自己处理信令交换（Signaling）、ICE 穿透、重连逻辑。

### B. 强大的分布式集群能力

  LiveKit 原生支持基于 Redis 的水平扩展。当单台服务器无法承载更多用户时，你可以轻松部署集群，系统会自动处理跨节点的房间路由。

### C. 现代网络协议支持

  HTTP/3 (WebTransport): 它是首批支持 WebTransport 的 RTC 框架，能有效缓解网络抖动带来的卡顿。
  WHIP/WHEP: 全面拥抱标准化，支持 OBS 等专业设备直接接入。

### D. AI 优先的设计 (AI-Native)

  LiveKit 是目前 OpenAI 和许多 AI 初创公司首选的底层框架。其内置的 VAD（语音活动检测） 和 流式处理 机制，解决了 AI 交互中“打断”和“延迟”的两大难题。

## 数据流向全景图

  一个典型的 LiveKit 通话流程如下：

  发布端 (Publisher): 采集音视频 -> 编码 (Opus/H.264/VP8) -> 通过 UDP 发送到 SFU。
  服务端 (LiveKit Server): 接收数据 -> 查找订阅者 -> 检查订阅者带宽 -> 转发 对应分辨率的数据包。
  接收端 (Subscriber): 接收 UDP 数据 -> 解码 -> 渲染。

## 部署与成本

  开源自托管: 你可以完全免费地在自己的服务器上使用 Docker 部署，没有任何功能限制。
  LiveKit Cloud: 官方提供的托管服务，适合不想维护基础设施、追求全球加速的企业。

## LiveKit 适合做什么？

  多人会议/教育: 类似 Zoom 的多人音视频。
  AI 语音助手: 具备打断功能、极低延迟的对话机器人（类似 GPT-4o 语音模式）。
  元宇宙/游戏: 空间音频支持，根据距离远近自动调节音量。
  实时直播: 亚秒级的超低延迟直播。
