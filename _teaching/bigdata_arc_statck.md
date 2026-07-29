---
title: "大数据架构栈演进"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/bigdata_arc_statck
date: 2026-07-29 13:30:00
---

# 简介

  10年前，hadoop 刚出来的时候，各种牛逼，技术组件各种组合，各种重型架构，会一点hadoop就感觉人上人了，哈哈哈。
  几年前FDAP（Arrow Flight + DataFusion + Arrow + Parquet） 架构栈，出现了一大堆自研数据库，数据仓库，数据湖，哈哈哈，各种种基于场景自研，哈哈哈， 懂得都懂，各种国产，各种爱国。
  最近 parquet，arrow，iceberg，polaris，ossie 的架构栈，越来越轻了，越来越清晰了，哈哈哈。也更统一了。傻瓜式的操作。哈哈哈。

  这里记录下，具体的细节自己ai及啃文档。


## 核心组件流程

[ 顶层：业务与 AI 语义对齐 ]  --->  Apache Ossie (统一指标与大模型上下文)
        ▲
[ 消费层：交互式多维查询 ]    --->  Apache Trino (高性能分布式 SQL 查询引擎)
        ▲
[ 计算与流批一体引擎 ]        --->  Apache Flink (实时流式计算、CDC 入湖)
        ▲
[ 存储与表格式层 ]            --->  Apache Iceberg / Apache Paimon (湖仓表格式 / 事务层)
        ▲
[ 元数据与权限治理 ]          --->  Apache Polaris (统一管理 Iceberg/Paimon 目录与鉴权)
        ▲
[ 底层物理与内存格式 ]        --->  Apache Parquet (磁盘列式存储) + Apache Arrow (内存列式计算)



















