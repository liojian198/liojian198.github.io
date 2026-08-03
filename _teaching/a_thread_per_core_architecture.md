---
title: "a thread-per-core architecture"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/a_thread_per_core_architecture
date: 2026-08-03 21:30:00
---

# 简介

  以前对媒体流，信息流开发的时候，为了高性能， 都会说 cpu亲和性，减少上下文切换。但是一直都是理论的，后面看到k8s里面有node亲和性。就是那种各种性能优化到极致，现在接触的框架，发现做的都是这个套路，
  来优化性能，所有的东西都有适合的场景，哈哈哈。

最近看 webrtc sfu 服务器代码， pulsebeam 也是往这块的优化。 加上之前看我们的撮合代码也是用了一样的架构设计，cpu亲和性。

# a thread-per-core

  Thread-per-core（每核一线程 / 单线程每核模型） 是现代高性能计算、高频交易（HFT）、游戏服务器以及分布式基础设施（如 ScyllaDB、Seastar 框架、Envoy 部分模块）中推崇的顶级并发架构。

  与传统的“多线程抢同一个任务队列（Worker Pool / Thread Pool）”或者“来一个请求开一个线程”完全不同，Thread-per-core 彻底颠覆了多线程并发的范式。

## 核心思想：什么是 Thread-per-core？

  简单来说：把物理 CPU 核心（Core）当作独立的计算机来用。

  1. 1:1 严格绑定：系统有几块物理 CPU 核心，就启动固定数量的工作线程（Worker Thread），并通过 CPU 亲和绑定（CPU Pinning），让每一个线程死死钉在一个专属的 CPU 核心上。
  2. 拒绝线程池抢任务：传统线程池里，所有任务丢进一个公共队列，多个线程去抢，会产生巨大的锁竞争。而在 Thread-per-core 中，没有全局公共任务队列。
  3. Shared-Nothing（无共享架构）：每个核心拥有自己独立的内存区域、独立的状态、专属的队列。核心与核心之间默认不共享任何可变状态，彻底告别锁（Mutex / SpinLock）。

## 为什么要用 Thread-per-core？（解决传统多线程的痛点）

  传统多线程（如 Thread Pool）在面对极高并发时，会遇到四大性能杀手：

  1. 上下文切换（Context Switch）开销：OS 调度器会让成百上千个线程在有限的 CPU 核心上轮流切换，每次切换都要保存寄存器、陷入内核，白白浪费大量 CPU 周期。
  2. Cache 抖动（Cache Thrashing）与伪共享（False Sharing）：多线程频繁抢夺同一块内存数据时，CPU 硬件的 MESI 缓存一致性协议 会疯狂失效，核心之间不断通过底层的互连总线同步 L1/L2 缓存，导致延迟暴增。
  3. 锁竞争（Lock Contention）：只要有多线程并发读写共享数据，就必须加锁。一旦发生锁冲突，线程就会从用户态挂起进入内核态（阻塞），延迟直接从“纳秒级”恶化到“微秒/毫秒级”。
  4. 不确定的长尾延迟（Tail Latency / Jitter）：因为线程随时会被 OS 调度走，你永远不知道一段代码从开始执行到结束中间被耽误了多久，这在追求极低延迟的系统中是致命的。

## a Thread-per-core 是如何运作的？（经典架构设计）

以一个高性能网络服务或撮合引擎为例，Thread-per-core 的典型工作流如下：

1. 网关与网络收包（NIC RSS）：
   
现代网卡支持 RSS（Receive Side Scaling），网卡硬件会根据 TCP 的四元组（源IP、源端口等）进行哈希，把不同客户端的连接均匀分发到网卡的多个硬件接收队列（Rx Queue）。

2.专属核心处理：

每一个 CPU 核心绑定一个网卡硬件队列（或者通过内核绕过技术如 DPDK / Solarflare EF_VI），直接从网卡的特定队列收包、解析、处理业务。

3. 跨核通信（如果必须）：
   
  如果 Core 1 处理的请求需要交给 Core 2，它们之间绝对不加锁，而是通过前面提到的无锁环形队列（Lock-Free Ring Buffer / SPSC 队列）进行异步消息传递。

## 它的优缺点是什么？

  优势：
  
  极致的性能与吞吐量：几乎零上下文切换、零锁竞争、极高的 L1/L2 缓存命中率。

  极佳的确定性（Determinism）：延迟非常平稳，没有突发的长尾延迟抖动。

  代码心智模型简单（单线程内）：在单个核心内部，所有的业务逻辑都在一个单线程里跑，完全不需要考虑线程安全、死锁、并发竞态，写业务代码就像写单线程程序一样爽快。

  挑战与局限：
  
  负载不均（Load Imbalance）：如果某个核心分到的任务特别重（比如某个热门交易对成交量暴增），该核心会被打满（100%），而其他核心却很闲。系统必须依赖精细的任务窃取（Work-Stealing）或动态分片重平衡来解决。

  无法利用多核并行完成单一复杂任务：一个大任务如果无法被拆分并塞给单核处理，它就无法享受多核加速（因为它被限制在单核内）。因此它更适合高并发、可分片（如按用户、按连接、按交易对分片）的业务场景。

  https://www.youtube.com/watch?v=iwRaNYa8yTw  撮合描述视频

# o_uring，eBPF RSS ，OOP vs ECS 

  这三组技术代表了现代高性能系统开发（无论是低延迟网关、金融撮合引擎、高性能网络服务器还是游戏引擎）中，突破传统软件架构瓶颈的底层硬核优化哲学。

  它们的核心指向完全一致：消除现代计算机体系结构中的“隐形杀手”——系统调用开销、内存复制、跨核缓存一致性抖动（Cache Thrashing）以及非连续内存带来的 Cache Miss。

## io_uring：告别系统调用与内存复制

  1. 传统模型的痛点传统的 Linux I/O 模型（如 read/write、epoll）采用同步系统调用。

     陷入内核（Syscall Overhead）：每次发起 I/O 操作，应用层必须从用户态（User Space）通过 trap 切换到内核态（Kernel Space），CPU 寄存器保存、上下文切换，带来几百个时钟周期的白白浪费。
     冗余内存拷贝（Memory Copy）：标准 socket 读写通常涉及多次内存拷贝（网卡 $\rightarrow$ 内核缓冲区 $\rightarrow$ 用户空间缓冲区），极大地消耗了内存带宽。

  2. io_uring 的颠覆之处
     
    Linux 5.1 引入的 io_uring 彻底重构了这一逻辑，基于共享内存的环形队列（Submission Queue (SQ) 和 Completion Queue (CQ)）：

    零系统调用（Kernel-Bypassing via Ring Buffer）：应用层只需往 SQ 里面塞入一个 I/O 请求（如 IORING_OP_READ），然后直接推进 Tail 指针。内核在另一端直接消费，全程不需要执行 syscall 指令。

    Zero-Copy / 投递合并：支持注册缓冲区（Registered Buffers）和 ring 方式的零拷贝投递，消除了用户态与内核态之间的数据搬运，把吞吐量和延迟推向极限。

## eBPF RSS Steering：消除跨核数据包分发

1. 传统模型的痛点
   
  在多核服务器接收高并发网络包时，网卡默认的 RSS（Receive Side Scaling） 硬件分发通常只基于报文的 IP/Port 四元组做简单 Hash。
  
  痛点：如果同一个客户端的多个请求、或者属于同一个应用会话的报文被 Hash 到了不同的 CPU 核心上，就会引发跨核通信。
  核心 A 收到包后，必须通过无锁队列或者锁把数据交给负责处理该会话的核心 B，导致严重的 Cache 污染 和流水线停顿。

 2. eBPF RSS Steering 的威力

  通过在网卡驱动层或 TC（Traffic Control）挂载 eBPF（Extended Berkeley Packet Filter）程序：
  
  精准可编程路由：eBPF 可以直接检查数据包的深层 payload（例如应用层协议头里的 Session ID、User ID 或交易对 Symbol），并利用 eBPF Flow Director / 动态 RSS 重定向，在网卡刚收到包的微秒级瞬间，直接将特定会话的报文精准投递（Steering）到处理该会话的专属 CPU 核心（Thread-per-core 架构中的目标核心）上。  
  效果：从物理网卡入口就实现了“业务闭环”，彻底消除了跨核转发和缓存抖动。

## OOP vs. ECS：极致的 CPU 缓存行为优化

  1. 传统 OOP（面向对象编程）的内存

     灾难在传统的面向对象设计中，一个实体（比如一个 Player 或 Enemy 对象）通常把自己的所有属性（Position、Velocity、Health、RenderMesh 等）和方法打包写在一起。
     内存布局（AoS - Array of Structures）：当你在堆上连续创建多个对象时，内存中呈现的是 [Obj1(Pos, Vel, Health)][Obj2(Pos, Vel, Health)]...。
     Cache Miss 地狱：当系统只需要更新所有怪物的 Position 时，CPU 必须把整个庞大的对象结构（包括根本用不到的 Mesh 和 Health 字段）全部加载到 CPU 缓存行（Cache Line，通常 64 字节）里。这导致宝贵的 L1/L2 缓存被大量“无效数据”占满，引发疯狂的 Cache Miss。

  2. ECS（Entity-Component-System）与面向数据设计（DOD）

     ECS 彻底拆散了 OOP 的封装，转向 SoA（Structure of Arrays）

     架构：

       纯数据组件（Components）：所有对象的 Position 紧凑地连续存放在一个大数组里；所有对象的 Velocity 紧凑存放在另一个数组里。

       系统（Systems）批量处理：当运动系统（Movement System）工作时，它只按顺序线性扫描 Position 和 Velocity 数组。

       硬件级缓存友善（Cache-Friendly）：由于数据在内存中绝对连续，CPU 的硬件预取器（Hardware Prefetcher）会提前把后续数据加载到高速缓存中。CPU 可以像流水线一样极速吞吐数据，没有任何指针跳转的等待（Pointer Chasing）。

## 三者的关系

  把这三者结合起来看，它们描绘了现代极高性能系统的底层演进方向：

  网络层：用 io_uring 消除系统调用与内存复制。
  
  传输层：用 eBPF RSS Steering 确保数据包直达目标 CPU 核心，不产生跨核干扰。
  
  应用/计算层：用 ECS（数据面向设计） 替代 OOP，确保 CPU 在处理业务逻辑时，内存访问永远是连续的、零缓存失效的。


  webrtc sfu 服务器的开发 https://pulsebeam.dev/blog/moving-to-thread-per-core
  游戏：bevy
  java： Affinity




     
