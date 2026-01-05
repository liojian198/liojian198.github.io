---
title: "ffmpeg简介"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/ffmpeg
date: 2026-01-05 13:07:00
---

# 概述

FFmpeg 是音视频领域的“瑞士军刀”，是一个极其强大的开源多媒体框架。它几乎支撑了全球 90% 以上的音视频处理需求，包括 YouTube、Netflix 以及你手机里的各种剪辑软件。
简单来说，FFmpeg 可以实现：录制、格式转换、音视频剪辑、滤镜处理、流媒体转发。

https://github.com/leandromoreira/ffmpeg-libav-tutorial/blob/master/README-cn.md

wasm: https://github.com/ffmpegwasm/ffmpeg.wasm

## FFmpeg 的核心组成部分

FFmpeg 不仅仅是一个工具，它由几个核心库和三个命令行工具组成：

命令行工具
ffmpeg： 核心转换工具。负责转码、切片、合并等几乎所有处理任务。

ffplay： 一个基于 SDL 和 FFmpeg 库的简单媒体播放器。

ffprobe： 多媒体流分析器。用于查看音视频文件的详细元数据（分辨率、比特率、编码格式等）。

底层核心库

libavcodec： 提供音视频编解码器（如 H.264, H.265, AAC, opus）。

libavformat： 处理容器格式（如 MP4, MKV, FLV）。

libavfilter： 提供各种滤镜处理（如去噪、缩放、加水印）。
