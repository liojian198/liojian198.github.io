---
title: "全息盒子和数字人"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/full_box_digital_people
date: 2026-06-30 14:30:00
---

# 简介

  在3D这个方向， 有很多东西要做， 哈哈， 我们今天结合我们的项目，讲下全息3D盒子和数字人涉及到的技术

# 技术

## 音频驱动口型同步

  在通用平台上（如 Linux、PC、Android 或高性能 SoC），实现音频驱动口型同步的核心流程可以概括为以下三层架构：

### 1. 信号处理层 (Signal Processing)

  这一层负责将音频波形转变为可用的数值。
  
  VAD (Voice Activity Detection)： 首先识别当前音频段是否为有效语音。使用 WebRTC VAD 或 Silero VAD，这些库非常成熟，能有效过滤背景噪音，避免模型在无声时乱动。
  
  频谱特征提取 (Mel-Frequency Cepstral Coefficients, MFCC)： 不要只看音量（振幅）。为了实现“口型”同步（而不仅仅是“张嘴”），需要分析 MFCC 特征。MFCC 能较好地模拟人耳感知，通过它可以区分不同的元音（例如“a”、“o”、“i”对应不同的嘴型参数）。
  
  实时音量归一化： 使用 AGC (自动增益控制)，保证在不同距离或录音环境下，模型嘴部动作的张开程度趋于一致，不会出现声音大时动作剧烈，声音小时几乎不动的情况。

### 2. 同步算法层 (Synchronization Strategy)

  这是口型自然与否的关键，涉及如何平滑处理参数。

  参数映射与插值 (Interpolation)：
  
  线性插值 (Lerp)： 在每一帧之间进行平滑过渡，防止参数突变。
  
  低通滤波器 (Low-Pass Filter)： 对开口度参数进行低通滤波，模拟人体肌肉运动的物理惯性，避免嘴部产生“高频颤抖”。
  
  音素映射 (Phoneme Mapping)：
  
  简单方案： 将 MFCC 特征映射为几种基础形状（Closed, Small Open, Wide Open）。
  
  进阶方案： 使用简单的 DNN (深度神经网络) 模型（如 DeepSpeech 的前端或轻量级口型生成网络 Wav2Lip 的变种）。你输入音频特征，模型直接输出几组控制参数（Visemes）。

### 3. 控制与驱动层 (Control & Driver)

  将算好的参数传给你的虚拟形象。

  协议规范：
  
  如果控制的是 Live2D，通常使用 Cubism SDK 的参数接口（如 ParamMouthOpen）。
  
  如果是 3D 模型（Unity/Unreal），通常通过 BlendShape (形态键) 来控制，例如控制 mouth_open、mouth_smile、mouth_shape_o 等维度。
  
  延迟匹配 (Latency Alignment)： 这是最容易被忽略的一点。音频播放往往有 Buffer。你必须在口型生成时，增加与音频播放缓冲区等长的延迟，确保视觉上的口型比声音稍微慢几毫秒，或者完全同步，绝对不能比声音快。

### 4.架构推荐

  如果你需要一套可落地的技术栈，建议采用以下组合：
  
  音频前端： FFmpeg (解码) + WebRTC VAD (静音过滤)。
  
  核心推理/计算：
  
  轻量级方案： 使用 TensorFlow Lite 或 ONNX Runtime 加载一个极小的口型生成模型。
  
  计算： 每 20ms 生成一次嘴部形状参数。
  
  渲染驱动：
  
  通过 OSC (Open Sound Control) 协议或者 ZeroMQ 将参数传输给渲染端。这种方式解耦了音频分析进程和模型渲染进程。

## 面部生成（DiT/Unet） 

## 动作/表情控制

## 推流/渲染后处理
