---
title: "graph_rag 详解"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/graph_rag
date: 2026-01-26 15:14:12
---

# 简介

  https://microsoft.github.io/graphrag/

  https://github.com/microsoft/graphrag

  https://github.com/DEEP-PolyU/Awesome-GraphRAG

  https://github.com/ChristopherLyon/graphrag-workbench

  https://www.microsoft.com/en-us/research/blog/graphrag-improving-global-search-via-dynamic-community-selection/

# 目前主流的 GraphRAG 竞品及替代方案

## 核心竞品（功能对标类）

  这些项目在架构上与微软 GraphRAG 类似，但在性能、成本或易用性上有所突破。

### A. Perplexica (开源版 Perplexity)

  特点：它不仅仅是 RAG，更是一个完整的 AI 搜索引擎。它集成了图索引能力，擅长处理联网搜索后的信息聚合。

  优势：实时性极强，比微软 GraphRAG 更轻量，适合构建对话式搜索。

### B. LangGraph (LangChain 生态)

  特点：这不仅仅是一个库，而是一个构建“循环代理（Iterative Agents）”的框架。

  优势：微软 GraphRAG 流程相对固定，而 LangGraph 允许你自定义图遍历的逻辑。你可以编写极其复杂的“思考路径”，让 AI 在知识图谱中多步跳转。

### C. Kotaemon (全能型开源 RAG)

  特点：一个专为终端用户设计的开源 RAG UI 框架。

  优势：它集成了 GraphRAG 索引功能，且拥有非常友好的可视化界面，支持本地 LLM（通过 Ollama），极大地降低了部署门槛。

## 图数据库原生方案（底层优势类）

  这类竞品由传统图数据库厂商推出，优势在于对超大规模图数据的查询效率（高性能查询）。

### D. Neo4j GraphRAG (Python SDK)

  特点：作为图数据库的老大，Neo4j 推出了专门的 Python 包。

  核心逻辑：它提倡“向量+图”的双重检索。它不依赖微软那种复杂的预处理摘要，而是通过 Cypher 语言实时检索相关子图。

  适用场景：数据量极大的企业级应用（亿级节点）。

### E. NebulaGraph (Graph RAG 策略)

  特点：国产图数据库之光，在分布式存储上优势明显。

  优势：它开源了 llama-index-networks 相关的集成，特别强调在处理具有高度复杂关联的结构化数据时的稳定性。

### F. FalkorDB

  特点：主打低延迟的图数据库（基于 Redis 思想）。

  优势：它针对 LLM 的知识提取做了原生优化，推理速度极快，适合对响应时间要求严苛的场景。

# 技术路线对比表

|维度|微软 GraphRAG|LangGraph|Neo4j / Nebula|Perplexica|
|--|--|--|--|--|
|核心逻辑|社区发现 + 层次摘要|代理工作流控制|实时路径查询|联网搜索 + 语义图|
|擅长场景|全局总结、宏观分析|复杂逻辑推理、多步任务|精确关系检索、大规模数据|实时问答、知识发现|
|部署难度|高 (索引成本大)|中 (需编写代码)|高 (需维护数据库)|低 (开箱即用)|
|开源状态|完全开源|完全开源|部分组件开源|完全开源|
