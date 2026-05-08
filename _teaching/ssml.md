---
title: "SSML 标记语言"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/ssml
date: 2026-05-08
---

# 简介

  最近在做TTS相关的内容，发现有个规范，让语音合成有丰富的表现力和情绪达到极致， 大家都想一次书写，各家TTS都能使用，并且不会走样，哈哈。
  就看到各家支持SSML标记语言，但是各家有各家的自定义的内容，但是规范只有部分内容，要用还得自己去适配这块的内容。 记录下这块的东西，后续使用的时候，再看看这块的内容情况。

  其实AIGC 方向， 结构化的操作，还是很重要的，要不然生成的内容， 哈哈哈哈，谁用谁知道， 毕竟transformer的多头注意机制，还是得去框定相关的attention， 要不然就哈哈哈， 谁用谁知道。

  现在不管是文字生成图片，代码生成，图生视频，其实都离不开结构化的内容，对模型进行框定，这样跨模型执行效果是差不多的，这里除非模型不支持， 否则还是可以的。大家都想一劳永逸，哈哈，这道路比较远。

  我们现在的图生视频， 都是瞎写，想到那，写到那，结果生成的视频可想而知， 所以现在就等结构化的内容出来了， 还得去摸索哈哈哈。

# 图片生成视频结构化提示词

## 1. Sora Prompting 范式 (Standard Structured Prompting)

  这是目前开源社区（如 GitHub 上的各大 Sora 复现项目）最通用的结构。它将自然语言解构为属性字典。
  
  核心结构：通常包含 Subject（主体）、Action（动作）、Environment（环境）、Camera（镜头）和 Style（风格）。
  
  开源体现：在 Open-Sora 或 Sora-WebUI 等项目中，后端逻辑通常会将用户输入的简单词汇通过 LLM 转换为这种结构。

## 2. MovieFlow / MovieFactory (时序剧情脚本协议)

  针对“长视频”或“有剧情视频”的生成，开源界倾向于使用一种类似 JSON-Story 的协议。

  特点：它不仅描述画面，还描述时间轴（Timeline）。

  结构示例：

``` JSON
{
  "storyboard": [
    { "timestamp": "00:00", "shot": "Extreme Long Shot", "content": "A robot waking up." },
    { "timestamp": "00:05", "shot": "Close-up", "content": "Eyes glowing blue." }
  ]
}
```

开源代表：MovieFactory (由新加坡国立大学等团队开源)，它定义了如何将文本脚本分解为连续的视频帧生成指令。

## 3.Prompt Travel / Keyframe Strings (AnimateDiff 标准)

在 SD (Stable Diffusion) 动画社区中，Prompt Travel 是一套非常成熟的“伪协议”。它通过特定格式的字符串来控制视频随时间演变的过程。

格式要求："帧数": "提示词"。

示例：

Plaintext
    "0": "a boy standing in the rain",
    "32": "a boy running under the sun",
    "64": "a boy flying into the sky"
    ```
*   **开源工具**：**AnimateDiff** 和 **ComfyUI**。这是目前可控性最强、插件生态最丰富的开源视频生成工作流，它利用这种时序映射来实现平滑转场。



## 4. **AIGC-JSON / VideoGen-JSON**
这是一些开发者尝试统一图像和视频生成参数的开源尝试，旨在建立一套跨模型的中间语言（IR）。

*   **目标**：无论底层是调用 Runway、Sora 还是本地的 Stable Video Diffusion，前端只需发送统一的 JSON 协议。
*   **包含参数**：`motion_bucket_id` (运动幅度)、`fps`、`noise_augmentation` 等物理参数。

---

## 5. **Scenario-based Tags (DSL)**
在你的 **XiaoZhi AI** 项目中，如果你需要更极简的控制，可以参考 **CogVideoX** 等项目中的 Tag 协议。

*   **核心逻辑**：使用特定的 **Trigger Words**（触发词）。
*   **开源实践**：通过训练微型 LoRA 模型，将复杂的动作（如“挥手”、“奔跑”）打包成一个短的标记符，在提示词中直接引用。

---


1.  **如果你追求极致的可控性**：参考 **AnimateDiff 的 Prompt Travel** 格式，它能让你精确定义第几秒发生什么。
2.  **如果你是调用云端 API (如阿里云 VEO)**：建议自建一套 **JSON 映射层**。将用户的语音指令先交给 LLM 转换成如下结构，再拼接成最终 Prompt：
    *   `{"subject": "...", "motion": "...", "camera": "..."}`
3.  **开源搜索关键词**：可以在 GitHub 搜索 `Awesome-AIVideo-Directing` 或 `Video-Prompt-Engineering` 获取最新的 Prompt 模板库。
