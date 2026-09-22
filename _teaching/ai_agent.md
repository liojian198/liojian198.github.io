---
title: "ai_agent 框架介绍"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/ai_agent
date: 2026-09-23 01:30:00
---

# 简介

  最近面试，看到各种找agent的，描述里面还是停留在提示词，上下文这些，其实对于企业级的agent来说，还是得有harness的。所有对比看了下目前主流的agent框架，各自有各种的优点，缺点。
  
  https://langfuse.com/blog/2025-03-19-ai-agent-comparison
  https://strandsagents.com/docs/learning/how-agents-really-work/

# 有哪些？

  LangGraph、OpenAI Agents SDK、Claude Agent SDK、Google ADK、Pydantic AI、CrewAI、Strands Agents、Mastra、Vercel AI SDK 以及 Microsoft Agent Framework 

  llm 是stateless的， 能输入参数也就那几个，主要有： 系统Prompt Engineering， tool execution，用户输入，推理。 模型就这些东西， 但是对于企业级的应用来说，不可能每次都手动拼装这些吧。
  业内就出现了agent，agent=llm+harness。这个是aws相关文章看到了。 所以harness主要包括： tools, context management, lifecycle hooks, memory, sessions, evals, and observability。
  这几样加起来做出企业级的应用还是比较方便的，但是作为AI agent底座 还要考虑安全等问题。

  记忆的话，我之前项目使用的是 memorylaker 这家的， 像本体这块，我们也是有涉及的，只是面向的场景不一样。

## 大概分类

  根据架构设计与适用场景，可以将这 10 个框架划分为五大阵营：
  强图流控与复杂状态机： LangGraph, Microsoft Agent Framework (MAF)
  类型安全与极简数据驱动： Pydantic AI
  大模型厂商原生 SDK： OpenAI Agents SDK, Claude Agent SDK (Anthropic), Google ADK (Agent Development Kit), Strands Agents (AWS)
  角色扮演与任务协同： CrewAI   
  Web 全栈与框架集成： Vercel AI SDK, Mastra


## 1. LangGraph   

  特点： 抛弃了传统 LangChain 的黑盒 Chain 抽象，使用显式的“节点 (Nodes)”和“有向边 (Edges)”来管理状态。具备行业最成熟的 Checkpointing（检查点机制） 和 Time-Travel（时间旅行调试）。   
  什么时候选：业务逻辑有严格的 SOP（标准作业程序），不允许 Agent 自由漫无目的地“瞎想”。需要强人工干预（Human-in-the-loop），比如 Agent 算好退款方案后，必须等待管理员点击“确认”才继续流转。

## 2. OpenAI Agents SDK (前身 Swarm)

  特点： 极其轻量。核心抽象只有两样：Agent 和 Handoff（交接）。没有繁重的数据结构，完全依靠 LLM 自动将控制权交接给下一个 Agent。   
  什么时候选：全家桶采用 OpenAI (GPT-4o/o3-mini) 模型。追求极简代码，希望快速实现像“客服转接技术支持，技术支持转接财务”这种轻量级多 Agent 系统。

## 3. Claude Agent SDK (Anthropic)

  特点： 深度集成 MCP (Model Context Protocol) 协议与 Computer Use (电脑屏幕自动操作) 能力。   
  什么时候选：核心模型依赖 Claude 3.5 Sonnet / Claude 3.7 Sonnet。需要 Agent 跨环境调用各种标准化的 MCP 工具（如 PostgreSQL、GitHub、Slack），或执行跨软件的 GUI 界面自动化操作。  

## 4. Google ADK (Agent Development Kit)   

  特点： Google 主导开源的企业级 Agent 框架，内置全套 CLI、Web UI 调试面板 和评测套件 (Eval)。原生支持 Gemini 多模态双向音视频流 和 Artifact 长期记忆管理。   
  什么时候选：基础设施基于 Google Cloud (GCP / Vertex AI)。需要处理语音、视频等实时多模态交互，或者构建需要严格评测与安全鉴权的多 Agent 系统。

## 5. Pydantic AI

  特点： 由 Python 著名的 pydantic 团队打造。主打“Type-Safe（类型安全）”，将 Prompt、模型响应、工具调用与 Python 的 Type Hints 完美结合，并提供强大的 Dependency Injection（依赖注入）用于测试。   
  什么时候选：团队对 Python 代码质量、类型检查（mypy/pyright）和单元测试有极高要求。极其注重 Agent 输入输出的数据结构校验，不希望解析 JSON 时频繁崩溃。   

## 6. CrewAI

  特点： 高度拟人化。将系统抽象为“Agent（角色）+ Task（任务）+ Crew（团队）”。你用自然语言给 Agent 设定“高级架构师”、“Code Reviewer”等身份，它会自动按顺序或并行完成任务。   
  什么时候选：需要快速搭建灵感碰撞、内容创作或软件工程试错的“虚拟团队”。适合原型验证（PoC）阶段，用极其直观的方式向非技术人员展示多 Agent 协同。

## 7. Strands Agents

  特点： AWS 开源的模型驱动（Model-driven）Agent 框架，内置 Harness 控制环、代码沙箱（CodeAct 模式）和 OpenTelemetry 链路追踪。   
  什么时候选：运行环境部署在 AWS（Bedrock, ECS, Fargate, Lambda） 上。Agent 需要频繁在沙箱中编写并运行 Python 代码（CodeAct）来解决问题，同时需要企业级的安全防范。

## 8. Mastra

  特点：TypeScript/Node.js 生态下的全栈 Agent 框架。它把 Agent、Workflow（工作流）、RAG 知识库、Sync 引擎以及 Dev Tools 整合在了同一个 Node 框架中。   
  什么时候选：纯 TypeScript 团队，想构建一个后端 Agent 服务（不仅是前端组件，还包含工作流和向量数据库集成）。
  
## 9. Vercel AI SDK

  特点： 前端/全栈 UI 时代的绝对统治者。它不是一个纯 Agent 编排库，而是连接 LLM 与 UI 的桥梁（提供 useChat, streamUI 等）。   
  什么时候选：使用 Next.js / React 构建 Web 应用。需要 Agent 在对话框中动态生成组件（例如：Agent 提到股票，前端直接渲染一个交互式 K 线图表格）。

## 10. Microsoft Agent Framework (MAF)

  特点： 微软整合 Semantic Kernel 与 AutoGen 的新一代企业级 Agent 框架。原生支持 .NET (C#) / Python / Go，并深度打通 Azure OpenAI 和 Microsoft Foundry 生态。  
  什么时候选：企业的技术栈是 C# / .NET 生态，无法全面转向 Python/TS。需要在 Azure 云端部署具备企业级合规、可观测性（OpenTelemetry）与复杂图多 Agent 系统的场景。

## 选型决策流程图（Quick Decision Guide）

  请根据你的团队技术栈与核心业务诉求，快速查找对应的推荐框架：
  “我是 Next.js/React 前端开发者，需要把 AI 交互无缝渲染到网页上”$\rightarrow$ Vercel AI SDK（如果还需要纯 Node.js 后端工作流，选 Mastra）
  “我是 C# / .NET 企业级后端团队”$\rightarrow$ Microsoft Agent Framework“
  业务流程绝对不能错，需要人工确认、严格的图节点跳转和断点恢复”$\rightarrow$ LangGraph“
  我是 Python 严格主义者，必须要最强的数据类型安全 (Type-Safety) 和测试支持”$\rightarrow$ Pydantic AI“
  深度绑定特定云厂商生态”AWS 生态： Strands AgentsGoogle Cloud (GCP/Gemini)： Google ADK   Azure / Microsoft 365： Microsoft Agent Framework   “
  想极简快速地尝试多角色协同，或者依赖模型厂商原生功能”想用角色扮演组建虚拟团队： CrewAI
  极简 OpenAI 多 Agent 转接： OpenAI Agents SDK   
  需要 MCP 协议工具支持或电脑屏幕操作： Claude Agent SDK

  

  
