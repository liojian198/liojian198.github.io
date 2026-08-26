---
title: "使用rust时，遇到的问题"
collection: talks
type: "talks"
excerpt: ''
permalink: /talks/rust
date: 2026-08-26
toc: true # 启用目录
toc_label: "rust 问题及解决方案" # 目录标题
toc_sticky: true # 目录是否固定在侧边 (可选)
---

# 简介

  这里记录在使用rust的时候遇到没见过的东西或者bug， rust是系统语言，涉及的领域非常多。 我这就不分领域了。

# 具体疑问

## unwrap_or_else 

## 闭包，fnOnce,mutfn, fn

## async,await,future       

## [derive(Debug, thiserror::Error)]

## [error("agent graph: {0}")]

## pub(crate) session: Arc<S>, 这里的 pub(crate) 详解

## [cfg(feature = "webrtc-helper")]

## where 

pub fn build_router<S, B>(state: AppState<S, B>) -> Router
where
    S: SessionSource + 'static,
    B: AgentBrain + 'static,

## [tokio::main]


async fn main() {


## vscode debug 时，需要 gdb， 插件 CodeLLDB



