---
title: "alexa使用"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/alexa
date: 2025-09-20
---

# 概述

    在上班的时候，公司产品要接入alexa和home kit,发现有个大牛，在公司搞对接alexa 搞了1个月，连流程都搞不清楚，可能公司没有什么压力吧。不扯了，先上图，只有图，哈哈。

# alexa 对接全景交互时序图

<img src='/images/alexa/alexa.png'>

# alexa 用户系统与我们的用户系统打通

<img src='/images/alexa/tokens.png'>

    注意http code 的301使用。 这里不涉及APP里面弹起内置浏览器，然后app通浏览器里面拿code码的流程，不需要在APP端定义协议做这个事情。


# 大概的配置

 1. Alexa Developer Console。 后台设置，有话术，意图，lambda等的设置要注意。

# 现在描述硬件对接alexa

## AVS

    AVS (Alexa Voice Service) 是亚马逊提供的一套云端服务 API，它允许硬件制造商将 Alexa 的智能语音能力（如语音识别、自然语言理解、智能家居控制等）直接集成到自研的联网设备中。

    简单来说，通过 AVS，你的设备（如智能音箱、智能后视镜、甚至微波炉）就能像 Amazon Echo 一样“听懂”人话并做出回应。

## AVS 的核心工作流程

当用户对着集成了 AVS 的设备说话时，发生的流程如下：

    1. 唤醒检测 (Wake Word Detection)： 设备本地运行一个微型模型，检测“Alexa”这个关键词。

    2.音频捕获与流式传输： 一旦唤醒，设备采集用户语音，通过 HTTP2 协议将音频流实时发送至云端 AVS。

    3.云端处理： Alexa 云端进行 ASR（自动语音识别）和 NLU（自然语言理解）。

    4.下发指令 (Directives)： AVS 向设备发送 JSON 格式的指令（例如：“播放音乐”、“开启灯光”或“播报天气”）。

    5.响应执行： 设备解析 JSON，执行相应动作（如播放音频流或操作硬件引脚）。



## AVS 的关键技术组件

AVS Device SDK： 亚马逊官方提供的 C++ 库，负责处理连接管理、音频交互、指令解析和状态同步。它能显著降低开发难度。

Audio Focus Manager： 自动处理声音优先级（例如：当 Alexa 说话时，背景音乐会自动调小）。

Media Player： 用于处理云端下发的音乐流或语音合成（TTS）音频。

Alerts/Settings： 本地管理闹钟、定时器及设备设置的同步。

## 硬件开发要求

要在嵌入式设备（如 ESP32 或 ARM 芯片）上运行 AVS，硬件通常需要满足以下条件：

    音频前端 (AFE)： 必须具备降噪、回声消除 (AEC) 功能，以确保在嘈杂环境或播放音乐时仍能听到唤醒词。

    内存要求： AVS Device SDK 通常需要较多的 RAM（通常建议 50MB+，但在 RTOS 环境下有精简版 SDK 可支持更低配置）。

    安全认证： 必须支持 TLS 1.2 及以上加密协议。

## AVS 与 ASK (Alexa Skills Kit) 的区别

|维度|AVS (Alexa Voice Service)|ASK (Alexa Skills Kit)|
|--|--|--|
|侧重点|做硬件。赋予物理设备“耳朵”和“嘴巴”。|做软件/功能。为 Alexa 增加新技能。|
|用户角色|硬件工程师、嵌入式开发者。|后端开发者、AI 产品经理。|
|示例|制造一款内置 Alexa 的智能头盔。|开发一个“查附近垃圾分类”的语音功能。|
