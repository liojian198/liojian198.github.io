---
title: "时空运动特征"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/Spatio_temporal_motion_features
date: 2026-08-18 21:30:00
---

# 简介

  在做ai语音陪聊的时候，如果只是陪聊，没有感知外面事务的能力， 那设备没有什么吸引力， Dipal 设备目前做的可以的， 有一小部分外部联动，最难的是他们融了1000万， 
  但是产品一直不能批量生产，出不了货。 这块的各种场景的融合还是不成熟。

    ai 语音陪聊， 我从经典三段式架构，到模型多模态的live/realtime， 再到streamer的方式，才短短1年的时间。 但是这些都是聚焦在聊天和响应速度上，但是没有融合当前用户所在场景及交互的捕捉。
    大家叫的具身智能。


    https://github.com/facebookresearch/SlowFast

# 时空运动特征

    按照这个想法，我们慢慢发现时空运动特征这个方向。slowfast是之前在做 国家安防的项目 研究过的。 基于视频做人群，动作分析，还是得meta， 哈哈哈。不说多了， 

## slowfast 行为识别 

  SlowFast 是由 Meta（Facebook）AI Research 于 2019 年提出的经典视频动作识别与检测架构（发表于 ICCV 2019）。
  它的设计灵感来源于生物学中灵长类动物视觉系统的视网膜神经节细胞：视网膜中约 80% 的细胞是 P 细胞（Parvocellular cells），负责低时间分辨率、高空间细节的静态色彩和形状感知；
  另外约 20% 是 M 细胞（Magnocellular cells），负责高时间分辨率、快速变化的运动感知

SlowFast 网络正是通过这种“双路径、双速率”的设计，极大地提升了视频行为理解的准确率和计算效率。

## 核心架构设计

  SlowFast 网络由两个并行的分支（Pathway）组成，分别处理不同帧率的视频输入：

  1. 慢通道（Slow Pathway）
     
     核心职责： 专注于空间语义（Spatial Semantics），捕获场景中的物体、背景和精细的外观细节。工作方式：采用低帧率（Low Frame Rate）和大的时间步长（Temporal Stride，通常 $\tau = 16$，即每秒视频只抽样 2 帧左右）。
     输入的通道数较多（即网络较宽），以提取丰富的空间特征。可以使用任何标准的 3D 卷积骨干网（如 3D ResNet）。
  
  2. 快通道（Fast Pathway）

     核心职责： 专注于时间动态与运动（Motion Dynamics），捕获快速变化的动作（如挥手、击打、跑步）。
     工作方式：采用高帧率（High Frame Rate）（时间步长通常为 $\tau / \alpha$，其中 $\alpha$ 通常设为 8，即帧率是慢通道的 8 倍）。
     轻量化设计（Lightweight）： 为了防止高帧率带来爆炸性的计算量，快通道的通道容量（Channel Capacity）非常小（通常通道数是慢通道的 $\beta = 1/8$ 甚至更少）。
     它不在中间做时间池化（Temporal Pooling），以保留完整的时间保真度。


## X3D

原理： 同样是 Meta 提出。它通过在空间、时间、宽度、深度等多个维度进行“复合缩放（Compound Scaling）”，找到计算量和准确率的最优解。

优势： 比 SlowFast 更轻，在移动端硬件上极其友好，是目前工业界部署率最高的视频识别模型之一。







