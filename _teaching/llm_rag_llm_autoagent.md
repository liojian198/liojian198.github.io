---
title: "RAG,LLM,AGENT 演进及说明"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/llm_rag_llm_autoagent
date: 2025-09-04 19:50:12
---

# 简介 

  本文介绍了 RAG 演进、智能体模式及 KV 缓存等大模型关键技术。

# 传统RAG与Agentic RAG对比

  先上偷来的图

  <img src='/images/llm/ragvsagenticRAG.png'>

## 1. 传统RAG

### 核心流程

检索（Retrieval）：从固定知识库中检索与输入相关的文档片段（如BM25/向量检索）。

生成（Generation）：将检索结果拼接为上下文，输入大模型生成回答。

### 特点

  静态处理：检索与生成分离，无反馈循环。

### 局限性：

  检索结果质量直接限制生成效果；

  无法动态优化检索策略；

  多跳推理能力弱（需人工设计分步查询）。


## 2. Agentic RAG

### 核心思想

  将RAG流程赋予自主决策能力，通过智能体（Agent）动态管理检索与生成。

### 关键改进：

#### 动态检索：

  基于生成内容的反馈调整检索策略（如改写查询、多轮检索）；

  支持复杂查询的多跳推理（自动分解子问题并迭代检索）。

#### 任务感知：

  根据任务类型（问答、摘要等）选择检索工具或生成策略；

  可调用外部API或工具补充知识（如计算、实时数据）。

#### 自我验证：

  对生成结果进行事实性检查（如二次检索验证）、逻辑一致性评估。

# 五种经典的智能体设计模式

  上图 

  <img src='/images/llm/five_agent_patterns.png'>


1. Reflection Pattern（反思模式）

核心思想：智能体通过自我评估与迭代修正优化输出。

流程：生成结果 → 分析错误/不足 → 调整策略重新生成。



2. Tool Use Pattern（工具使用模式）

核心思想：智能体调用外部工具（如API、计算器、搜索引擎）扩展能力边界。动态选择工具并解析工具返回结果。



3. ReAct Pattern（推理+行动模式）

核心思想：结合推理（Reasoning）与行动（Action）的交互式决策。

流程：

Reason：分析当前状态（如“需要查询天气”）；

Act：执行动作（如调用天气API）；

循环直至解决问题。


4. Planning Pattern（规划模式）

核心思想：智能体预先制定分步计划再执行，而非即时反应。长期目标分解为子任务，动态调整计划。



5. Multi-agent Pattern（多智能体模式）

核心思想：多个智能体通过协作/竞争完成复杂任务。角色分工（如管理者、执行者）、通信机制（如投票、辩论）。

# 5大文本分块策略

 上图： 

 <img src='/images/llm/five_chunk_strategies_RAG.png'>

1. Fixed-size Chunking（固定分块）
核心思想：按固定长度（如 256 tokens）分割文本，可重叠（滑动窗口）。

优点：简单高效，适合常规 NLP 任务（如向量检索）。

缺点：可能切断语义连贯性（如句子中途截断）。

场景：BERT 等模型的输入预处理、基础 RAG 系统。



2. Semantic Chunking（语义分块）
核心思想：基于文本语义边界分块（如段落、话题转折点）。

实现：

规则：按标点（句号、段落符）分割；

模型：用嵌入相似度检测语义边界（如 Sentence-BERT）。

优点：保留语义完整性。

缺点：计算成本较高。

场景：精细化问答、摘要生成。



3. Recursive Chunking（递归分块）
核心思想：分层分割文本（如先按段落→再按句子）。

优点：平衡长度与语义，适配多级处理需求。

缺点：需设计分层规则。

场景：长文档处理（论文、法律文本）。



4. Document Structure-based Chunking（基于文档结构的分块）
核心思想：利用文档固有结构（标题、章节、表格）分块。

实现：解析 Markdown/HTML/PDF 的标签结构。

优点：精准匹配人类阅读逻辑。

缺点：依赖文档格式规范性。

场景：技术手册、结构化报告解析。



5. LLM-based Chunking（基于大模型的分块）
核心思想：用 LLM（如 GPT-4）动态决定分块策略。

方法：

直接生成分块边界；

指导规则引擎优化（如“将这段话按时间线拆分”）。

优点：灵活适配复杂需求。

缺点：成本高、延迟大。

场景：高价值文本处理（如医疗记录、跨语言内容）。

# 智能体系统的5个等级

  上图

  <img src='/images/llm/five_agentic_ai_systems.png'>


# 传统RAG vs HyDE

  上图

  <img src='/images/llm/rag_hyde.png'>

# RAG vs Graph RAG

  <img src='/images/llm/RAGvsGraphRAG.png'>
 
  RAG 直接检索文本片段，适合短平快问答；

  Graph RAG 利用知识图谱的结构化关系，更适合需要逻辑推理的任务（如“某药物的副作用机制是什么？”）。


# KV caching 

<img src='/images/llm/KVcaching.png'>