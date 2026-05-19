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


  
