---
title: "语言特性提案相关规范"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/12_factor_ai_agents
date: 2026-08-03 04:30:00
---

# 简介

  最近在看rust，wasm，各种看到资料，各种迷迷糊糊，描述的很片面，只是描述语法就这样，但是为什么这么设计，这么设计是为了解决什么问题， 哈哈。感觉神神奇奇的。 
  用java这么多年了， 基本上 有新特性第一反应是看jsr或者平时没什么事情，就去看jsr上有什么提议。这些东西还挺有意思的。 写rust写了一段时间了，发现这玩意在嵌入式，服务器，wasm方向还是很牛逼的。
  我见过嵌入式写C++各种内存问题，各种资源管理问题，还是那句，没见过这么水的嵌入式。哈哈哈。

# 编译原理

  语言嘛，最重要的是编译器， LLVM，哈哈哈，难啃的骨头。但是前端部分还是很简单的， 后端部分就傻了。

  所有的特性基本上是在前端完成的。具体举例rust

  rust 源码--->tokens--->ast--->HIR---->MIR--->LLVM IR --> ......

  深入一门语言还是得从编译原理入手，这样才会知道为什么这样，为什么那样。哈哈哈

编译原理后端 也有很多， 特别是在嵌入式领域，哈哈哈， 我们现在看 wasm比较多。哈哈哈哈哈

# 语言规范资料地址

java： jsr

rust： https://rust-lang.github.io/rfcs/， https://rustc-dev-guide.rust-lang.org/overview.html

go： https://github.com/golang/proposal#readme

wasm： https://github.com/WebAssembly/meetings/blob/main/process/phases.md， https://github.com/WebAssembly/meetings/blob/main/process/proposal.md， https://github.com/WebAssembly/proposals


python： https://peps.python.org/
