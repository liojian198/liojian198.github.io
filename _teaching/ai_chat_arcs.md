---
title: "陪伴ai几种架构介绍"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/ai_chat_arcs
date: 2026-01-05 16:19:00
---

# 概述

  目前工作中，接触到的陪伴ai架构主要有3种，陪伴的范围很广，其实也是包含数字人的。但是这次我们主要是聊chat架构，虽然这些架构也适合ai视频chat。

# 简介

  第一种: 传统型的架构模式
  第二种: S2S型的(全模态，包括：文本，音频，视频，图片的realtime接口)
  第三种: 基于直播类型架构实现。例如: livekit，Pion 这种类型 

## 传统型的架构模式

  之前的文章有介绍: < https://liojian198.github.io//teaching/ai_chat >

## S2S型的

  这类架构主要是将传统架构里面的aec，vad，asr，llm，tts 集成到模型里面去了， 通过模型的pipeline的方式来统一处理。结合serverless，边缘服务器的方式来统一调用模型提供的api。
  目前开源的S2S模型有： moshi 模型， 我在mac上跑了下， 直接烫到不行了。
  其它的: Amazon Nova Sonic，Azure OpenAI Realtime API，Gemini Live API，OpenAI Realtime API， Ultravox Realtime，xAI Grok Voice Agent API

  国内： https://help.aliyun.com/zh/model-studio/multimodal-products-overview?spm=a2c4g.11186623.help-menu-2400256.d_1_12_3_0.1c573386iF6dzo&scm=20140722.H_2922371._.OR_help-T_cn~zh-V_1

## 基于直播类型架构实现

  利用直播流到房间技术，chat ai 模拟一个客户端，进行ai处理，不仅限于聊天，ai视频机器人也是支持的，数字人，2D/3D模型也支持的。
