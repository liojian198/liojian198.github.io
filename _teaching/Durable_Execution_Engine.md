---
title: "持久执行引擎"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/Durable_Execution_Engine
date: 2025-09-18
---

# 概述

    执行引擎，我们经常见到，例如xxl-job, airflow,digester,flink等，但是持久化执行引擎很少见，但是我们在业务中经常用到，只是和业务耦合在一起了。所以最近看到一片文章: https://www.kai-waehner.de/blog/2025/06/05/the-rise-of-the-durable-execution-engine-temporal-restate-in-an-event-driven-architecture-apache-kafka/ 有对这个方向的抽象描述及实现原理描述，所以记录下。

    记得以前做金融系统的时候，我们也是有持久化引擎的概念的，只是实现的时候，耦合到代码里面了。通过持久化，mq，定时任务不断的拉起，重试，幂等的处理，转台的转移(saga)。我印象最深的是我们在风控系统里面抽象的更好，但是还是耦合到代码里面了，看到这个文章和涉及到的中间件，我发现当时的抽象还是比较水的，但是要抽象到文章涉及的级别，那要对整个技术框架和业务代码进行调整(架构设计都变了，把业务重新写，按照这框架的思维进行整合，这似乎是不可能的，一天几千万的现金流水，老板一句话：滚，你是找死嘛。).

#  持久执行引擎类型

## 嵌入型

    常见的有DBOS， 这个东西很新，基于PG的扩展。可以和supabase融合。
    deepwiki地址: https://docs.dbos.dev/architecture
    

## 业务代码与持久执行引擎分离型

    常见的有: restate,这个东西也很新。
    deepwiki地址: https://deepwiki.com/restatedev/restate

    deepwiki 是kevin通过llm 的agent生成的，做的还可以的，我们自己也是想通过agent来解析现有的技术，让人更容易理解上手。

## 业务代码与持久执行引擎融合型

    常见的有: temporal 这个东西收费的，但是做的很强大，可以做成模仿的对象，毕竟是这个很垂直领域的东西。

    deepwiki地址:  https://deepwiki.com/temporalio/temporal

