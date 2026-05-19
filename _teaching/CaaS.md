---
title: "Serverless Container Computing"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/CaaS
date: 2026-05-19 16:19:00
---

# 简介

  最近在做音视频服务部署设计的时候，我们在音视频这个方向，一般分控制面，数据面。 这种指令型和流型分离的方式也很常见，主要是为了方便扩容，缩容，这样对成本的收缩是有好处的，也方便处理。但是又不想去折腾k8s那套东西，
  其实不是不想折腾，k8s需要节点，ecs或者机器来支撑，不要告诉我，Serverless 容器集群也能部署k8s，这种套娃的方式，我想想都怕。我还是单独看下Serverless 容器怎么玩吧，现在云厂商绝大部分是支持这套东西都。

# Serverless 容器（Serverless Container） 或 无服务器容器计算（Serverless Container Computing）

  在云计算的模型分层中，它属于 CaaS（Container-as-a-Service，容器即服务） 演进到极致的终极形态。
  
## 1. 为什么叫“Serverless 容器”？

  这个术语精准地概括了它们的两个核心技术特征，将两大技术阵营完美融合：
  
  Serverless（无服务器）：指用户不需要购买、配置、管理、升级或扩容任何底层的虚拟机（如 ECS 或 EC2）。操作系统补丁、底层硬件维护、安全隔离全由云厂商搞定。计费完全按照容器实际消耗的资源（vCPU/内存）和运行秒数精准计算。
  
  Container（容器）：指它的运行载体是一个标准的 Docker / OCI 镜像。它不像 AWS Lambda 或阿里云函数计算（FC）那样限制你的代码结构和编写范式，它给你的是一个完整的、拥有自主生命周期、可以监听任意端口、支持多线程/多进程的纯粹 Linux 容器环境。

## 2. 行业标准与各大云厂商的对标命名
  
  在国际云原生计算基金会（CNCF）和各大云厂商的白皮书中，大家都围绕 Serverless Container 这一通用术语构建了自己的产品矩阵。你可以看这张行业对标表：

AWS (亚马逊云),AWS Fargate,Serverless Container

阿里云,ECI (Elastic Container Instance，弹性容器实例),Serverless 容器

微软云 (Azure),ACI (Azure Container Instances),Serverless Container

谷歌云 (Google Cloud),Google Cloud Run (偏向微服务托管),Serverless Container

华为云,CCI (Cloud Container Instance，云容器实例),Serverless 容器

腾讯云,TKE Serverless (原 EKS),Serverless 容器

## 3. 在 K8s 国际生态中的代名词

  如果你在 Kubernetes (K8s) 的开源生态里讨论 ECI 或 Fargate 的底层网络和调度对接，大家通常会使用另一个极其硬核的技术术语：
  
  虚拟节点（Virtual Kubelet）
  
  因为在 K8s 的世界里，原本一个 Node（节点）就必须是一台实实在在的服务器。但是为了接入 ECI 或 Fargate 这种“招之即来、挥之即去”的无服务器算力，微软和 AWS 联合开源了 Virtual Kubelet 规范。
  
  通过虚拟节点技术，K8s 大脑会把阿里云 ECI 资源池或 AWS Fargate 资源池伪装成一台拥有无限 CPU、无限内存的“虚拟服务器”。当有高并发流量进来时，K8s 不需要去买新的 ECS/EC2 节点，而是直接把 Pod 调度到这个“虚拟节点”上，在底层就会瞬间长出一个个 ECI 或 Fargate 实例。

## 4. Serverless 容器与lambda 有什么区别？

  简单来说：ECS 是 AWS 的容器编排“大脑”（指挥官），而 Fargate 是承载容器运行的“肉身”（打工人）之一。

### 一、 核心概念：指挥官与打工人的关系

  在 AWS 的生态里，要想把一个 Docker 镜像跑起来，你需要两样东西：
  
  编排系统（Control Plane）：负责管理容器的生命周期、扩容缩容、负载均衡、健康检查。这个“大脑”就是 ECS（或者 Kubernetes 生态的 EKS）。
  
  计算资源（Data Plane / Runtime）：容器总得消耗 CPU 和内存，这些硬件资源从哪里来？AWS 提供了两种完全不同的底层供你选择：ECI 虚拟机模式（即传统 ECS 部署） 和 Serverless 模式（即 Fargate 部署）。

### 二、 传统 EC2 模式 vs Fargate 模式的深度对比

  当你决定使用 ECS 作为大脑后，你需要决定容器落地的“肉身”是传统的 EC2 虚拟机 还是 Fargate（无服务器容器）。

  1. 传统 EC2 模式：自建机房式体验
  如果你选择 ECS + EC2：
  
  怎么运作：你需要自己去买几台虚拟服务器（EC2 实例），并给它们安装 AWS 的容器 Agent。这些 EC2 组成了你的“计算集群”。你的 Docker 容器就运行在这些你亲手管理的服务器上。
  
  你的职责：你需要负责这几台 EC2 操作系统（OS）的安全补丁更新、SSH 密钥管理、以及服务器本身的扩容（当服务器 CPU 满了，你要写脚本自动买新的 EC2 塞进集群）。
  
  2. Fargate 模式：纯粹的 Serverless 容器体验
  如果你选择 ECS + Fargate：
  
  怎么运作：你不需要管理、甚至看不到任何底层的服务器。 你只需要告诉 ECS：“请帮我把这个 Pipecat 容器跑起来，给它分配 2 核 CPU 和 4GB 内存。”
  
  底层内幕：AWS 会在自己的超级大资源池里，瞬间划出一个孤立的、微型的安全沙箱运行你的容器。容器一启动就开始计费，容器一关闭计费瞬间停止。
  
  你的职责：你唯一需要关心的就是你的 Dockerfile 代码和业务逻辑。底层的操作系统补丁、服务器扩容、硬件维护全部由 AWS 团队在幕后搞定。

## 5.Serverless 容器与lambda 的区别？

  虽然 AWS Fargate 和 AWS Lambda 都属于 AWS 旗下的 Serverless（无服务器） 计算大类，都具备“无需管理底层服务器、自动弹性扩容、按需付费”的特性，但它们的核心设计哲学、底层的运行机制以及适用的业务场景有着天壤之别。
  
  如果用一句话来概括它们的本质区别：
  
  Lambda 是“事件驱动的函数片段（Function-as-a-Service）”，代码只有被事件戳一下才会弹起来执行；而 Fargate 是“无服务器的常驻容器（Container-as-a-Service）”，它是一个完整的、拥有自主生命周期的轻量级操作系统沙箱。

### 核心维度生死对决

  1. 生命周期与运行超时（Lifetime）
AWS Lambda：短命鬼。单次请求的最大执行超时时间被死死限制在 15 分钟。如果你的任务超过 15 分钟没跑完，AWS 会在底层无情地直接强行掐断。它天然不适合用来维持长连接（如长达 1 小时的语音通话）或长周期的离线数据批处理。

  AWS Fargate：常青树。它没有任何超时限制。只要你不主动关闭它，或者你的系统没有崩溃，Fargate 容器可以在云端连续运行几天、几周、甚至几个月。它非常适合用来跑全双工的 WebSocket 服务器、WebRTC 媒介网关，或者标准的 Web 微服务。
  
  2. 触发与通信模型（Trigger & Protocol）
  AWS Lambda：被动事件驱动。它自己没有网络端口的概念，不能直接监听外界的 TCP/UDP 端口。它必须依赖上游事件（如 API Gateway 收到一个 HTTP 请求、S3 上传了一个文件、或者 DynamoDB 产生了一条新记录）来顺发“唤醒”它。
  
  AWS Fargate：主动常驻监听。它是一个功能完整的容器，拥有自己独立的虚拟网卡（ENI）和私有/公网 IP。它可以像普通服务器一样，使用 FastAPI 或 Netty 在内部直接监听 80、443、8080 等任意标准 TCP/UDP 端口，主动维护全双工长连接通道。
  
  3. 环境与硬件控制（Environment & Resources）
  AWS Lambda：高度受限的沙箱。虽然支持 Docker 镜像，但你无法控制底层的进程管理。你无法在 Lambda 内部安全地运行多个后台子进程，且最大内存限制为 10GB，不支持 GPU。
  
  AWS Fargate：完全可控的纯粹容器。你可以利用 ECS 编排，在一个 Task（任务组）里同时塞进多个协作容器（Sidecar 模式，如一个跑 Pipecat 主程序，另一个跑 Nginx 反向代理）。Fargate 拥有更广阔的计算资源规格，并且在部分 Region 原生支持了英伟达 GPU 加速。



