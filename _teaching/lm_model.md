---
title: "lm-微调"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/lm_model
date: 2025-12-23
---

# 概述

  一直在做llm，vlm，asr，tts这类的内容，各种延时，各种随机不确定，哈哈。 听到最多的就是调下参数，换个新模型或者直接上S2S，哈哈，借着最近没有那么多事情，大概看下联结派的产物大概有哪些内容和板块。

# 使用

  这块就略了，可以找个厂商的网页用用，或者直接用小智类的产品把概述里涉及的内容都体验一遍。这块就不细化了。

# 自己部署模型

目前我接触到的模型格式主要有3种： GGUF，MLX，core ML。

我有个mac电脑M4，32G内存的。

安装模型使用了 nexa，没用ollma。

使用nexa是因为它支持MLX架构，优化的效果还是比较好的。

## 注意： 在安装moshi 模型的时候，没有用nexa，直接就是下载，运行了个python脚本，电脑差点着火了。

# 解析生成的模型格式里面主要有哪些内容

  每个模型格式文件里面都包含如下内容：
  1. 权重： 本质上是成千上万的巨大矩阵。
  2. 分词器： 很关键的东西，没这个东西，模型就废了。我们称之为模型翻译官，负责将文字转换成数字，模型才能处理。

# 把训练好的原始模型（通常是 Safetensors 或 PyTorch 格式）转换成 GGUF 或 MLX，你需要使用不同的工具链

  MLX 转换更加简单，因为 Apple 官方提供了 mlx-lm 库，一行命令就能搞定。

  第一步：安装工具
  
  pip install mlx-lm

  第二步：转换与量化

  python -m mlx_lm.convert --hf-path /你的原始模型路径/ \
                         --mlx-path /输出路径/ \
                         -q --q-bits 4

  核心文件对应关系

  在转换过程中，工具会自动寻找并处理以下文件，请确保你的模型路径下包含它们：

|原始文件|	作用|	转换后的去向|
|:-|:-|:-|
|*.safetensors|	权重数据|被压缩并存入 GGUF 块或 MLX 张量中。|
|tokenizer.json|分词器|被打包进 GGUF 的 Header 或 MLX 的文件夹。|
|config.json|模型架构参数|被转换为 GGUF 的 KV 元数据或保留为 MLX 配置。|
|generation_config.json|	生成参数|	通常被 GGUF 忽略，MLX 会保留。|
  
