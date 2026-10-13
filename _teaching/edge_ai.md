---
title: "端侧模型"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/edge_ai
date: 2026-10-13 20:30:00
---

# 简介

  之前接触的模型基本上是服务器端的模型，或者端侧的视觉模型，最近发现大家卷到端侧llm/asr/tts， 这里的端侧可以是设备端esp32，stm32或者手机端。看具体的定义吧。就是模型运行时需要的资源很小。哈哈。

# 常见的有哪些？

## FunctionGemma 系列 (Google)

  特点：Google 推出的专为函数调用和工具集成设计的轻量级模型（如 FunctionGemma-270m）。

  定位：虽然参数量在几百 MB 级别（略大于 Needle 3），但其核心目标完全一致——放弃复杂的日常闲聊，专注在单次或多轮对话中精准解析用户的意图并输出符合 Schema 的 JSON 工具参数。

## Granite Code / Granite Micro 模型系列 (IBM)

  特点：IBM 推出的面向企业与边缘计算的微型基础模型（部分版本在 350M 以下）。

  定位：强调代码逻辑、精确指令遵循和结构化输出，非常适合在工业网关和受限计算环境中运行代码片段或结构化解析任务。

## Qwen-0.5B / Qwen-1.8B 系列中的 Function Calling 微调版 (阿里巴巴)

  特点：通义千问的小参数版本（如 Qwen2.5-0.5B-Instruct）。

  定位：虽然体量在 500M ~ 1GB 左右（比 Needle 3 略大），但由于其极其优秀的指令遵循能力和强大的多语言支持，常被开发者通过 LoRA 微调后部署在端侧，作为高精度的中文字面工具路由器。

## Needle 3（由 Cactus Compute 开发）

 1. Needle 3（由 Cactus Compute 开发）是面向端侧、嵌入式及物联网设备（如手机、穿戴设备、智能家居网关）设计的极轻量级专属基础模型。
  它故意舍弃了通用的闲聊和泛文本生成能力，将参数体量压缩至 8MB – 29MB（通过极其激进的量化如 CQ2-bit 与 Laddered 架构），
  专注于解决端侧 AI 中最核心的三个任务：工具调用（Tool Calling）、结构化数据提取（Structured Extraction）和文本嵌入（Embeddings）。

2. 极致轻量化与端侧推理运行时生态
除了特定名称的模型外，针对 Needle 3 所在的“MB级（< 50MB）”端侧极限窄带任务，整个技术生态正在向以下方向演进：

极低比特量化与专用架构（如 Ternary / 2-bit 模型）：

如同 Needle 3 采用约 2.125 bits/weight 的超低比特压缩（Cactus Quants）与 Simple Attention Networks (SAN) 架构一样，社区中越来越多的研究将极小模型（如 10M–50M 级别）通过三元量化（Ternary Quantization）压入微控制器（MCU）或智能手表内存中。

端侧多模态与语音集成小模型（如 Whistle 等）：

许多与 Needle 协同的边缘语音/意图一体化模型（如 Cactus 同步推出的 Whistle 语音转文字模型），支持将音频输入直接在本地解析并与 Needle 的工具路由表结合，实现端侧“一句话唤醒并执行设备控制”。


## SmolLM / SmolLM2-135M (Hugging Face)

  体积/参数：约 1.35 亿参数，经过极致量化（Q4/Q2）后体量可压缩至 70MB - 100MB 左右。

  特点：Hugging Face 专门为端侧设计的极小模型。其中 135M 版本在经过 Tool Calling / JSON Schema 微调后，非常适合部署在手机、树莓派等边缘设备上，进行本地指令解析与 JSON 导出。


## SLMEA / Micro-BERT 意图分类与槽位填充模型

  体积/参数：通常在 10MB - 30MB。
  
  特点：在工业界和智能家居网关（如 Home Assistant）中广泛应用。
  它们放弃了 Transformer 自回归生成，采用 Joint Intent Detection and Slot Filling（联合意图检测与槽位提取） 架构，将用户的口语指令（如“把客厅空调调到 26 度”）直接映射为 API 参数，毫秒级响应且零内存负担。

## Pico-LLM / TinyLlama-1.1B 极致裁剪版

  体积/参数：通过剪枝（Pruning）与网络层剪裁（Layer Pruning），将底层 Layer 裁剪至 4~6 层后量化，镜像大小可降至 50MB - 120MB。
  
  特点：只保留对 JSON 格式和函数定义的解析能力，专用于本地 Agent 的特定领域 Router（路由决策）。


## Whistle

  Whistle 是由 Cactus Compute 推出的一款超轻量级、专为端侧、嵌入式及边缘 AI Agent 设计的离线语音识别（ASR）与音频特征模型。它被设计为 Needle 3 等轻量级工具调用模型的“语音前端”，
  解决了物联网和边缘设备在无 GPU、低算力、弱网或断网环境下无法高效处理语音交互的痛点。

  极小的物理体积（16.9 MB）：通过 Cactus 专有的极低比特量化技术（Cactus Quants，2 至 4 bit），
  整个模型被打包进单一的 .cact 容器中，仅需数十兆内存即可加载。
  零 GPU 依赖（纯 CPU 极速推理）：运行在高度优化的 C++ 自研运行时上，利用 SIMD 硬件指令加速。在普通 CPU（如 Apple M 系列或嵌入式 CPU）上，
  首 token 延迟（TTFT）仅需 11.1 毫秒，解码吞吐量可达 1,319 tokens/秒。阶梯式解码架构（Laddered Decoder）：支持灵活调整解码层深度（从 2 层起步动态裁剪），
  开发者可以根据硬件的内存与功耗预算在精度和速度之间进行权衡。多语言与端到端协同：原生支持 7 种主要语言（英语、德语、法语、西班牙语、意大利语、荷兰语和波兰语）。
  完美对接 Needle 框架：音频经过 Whistle 编码后，可直接在内存中流转给 Needle 甚至跳过中间纯文本转换，直接完成语音 $\rightarrow$ 结构化工具调用（Audio-to-Tool-Call）。

  高精度转录（Transcription）：支持处理 16 kHz 单声道音频，单次推理最高支持 30 秒的连续语音输入。

  原生词级时间戳（Word Timestamps）：通过 Decoder 层的注意力权重直接输出精准到词级别的起始与结束时间，便于做长语音对齐与交互高亮。

  语音特征嵌入（Speech Embeddings）：支持仅提取 Encoder 阶段的语音特征向量（每 80ms 一帧），无需进行完整的文本转录，即可用于端侧声纹匹配、唤醒词验证或离线音频检索。

  关键字偏置（Keyword Biasing）：结合 Aho-Corasick 自动机，允许在解码时注入领域专有名词或设备代号白名单，大幅降低特定环境下的识别错误率。

  智能静音与幻觉过滤：在解码前置处理中加入静音检测，彻底避免了传统小模型在安静环境下产生“无中生有”的幻觉文本输出。

  
