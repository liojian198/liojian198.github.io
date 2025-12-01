---
title: "使用webflux时遇到的问题"
collection: talks
type: "talks"
excerpt: ''
permalink: /talks/webflux_problem
date: 2025-12-01
toc: true # 启用目录
toc_label: "使用webflux时遇到的问题及解决方案" # 目录标题
toc_sticky: true # 目录是否固定在侧边 (可选)
---

# webflux集成R2DBC的核心流程

  这是一个将 Reactor 编程模型和 Netty 线程模型结合起来的核心流程。Event Loop 执行 R2DBC 的过程，关键在于 I/O 等待时期的线程释放和回调执行的恢复。

  下面是 Event Loop 线程如何执行 R2DBC 数据库操作的详细步骤：

  Event Loop 执行 R2DBC 的七个步骤
  假设您的 WebFlux 控制器中已经定义了一个响应式流，并最终被 Netty 的 Worker Event Loop 线程接收并开始执行。

  1. 启动（订阅）信号到达

  线程： Netty 的 Worker Event Loop 线程 (reactor-http-nio-X)。

  动作： 响应式流的终点（通常是 WebFlux 框架本身）发出 subscribe() 信号。该信号沿着 Flux 或 Mono 的操作符链向上游传播，最终到达 R2DBC 驱动。

  2. R2DBC 发起非阻塞请求

    线程： 仍是 Event Loop 线程。

    动作： R2DBC 驱动接收到信号后，构造 SQL 语句并准备通过网络发送给数据库服务器。

  3. I/O 注册和发送

    线程： 仍是 Event Loop 线程。

    动作： R2DBC 驱动使用底层的 Java NIO 接口，将 SQL 请求数据非阻塞地发送到数据库 Socket 连接。

    关键： 驱动程序将该连接上的 Read 事件兴趣（等待数据库响应）注册到 Event Loop 关联的 I/O 多路复用器（Selector/Epoll） 上。

  4. 控制权交出（Event Loop 释放）

    线程： Event Loop 线程。

    动作： SQL 请求发送完毕，并且 Event Loop 已经完成了 I/O 注册。此时，数据库的响应还没有回来。Event Loop 线程不会阻塞等待，而是立即从 R2DBC 的代码中返回，回到它自己的事件循环主循环。

    结果： 此时，Event Loop 线程开始处理其他连接上的 I/O 事件（例如，处理另一个客户端的请求、写入响应数据等）。

  5. 内核通知（数据返回）

    线程： 无（操作系统内核）。

    动作： 数据库处理完查询后，将结果通过网络发送回来。操作系统内核检测到之前注册的 Socket 上有数据准备就绪。

    信号： 内核向 Netty 发送一个 就绪信号。

  6. Event Loop 唤醒和数据读取

    线程： Event Loop 线程被唤醒。

    动作： Event Loop 线程不再处理其他任务，而是响应内核信号。它执行 I/O 读取操作，从 Socket 中读取数据库返回的结果数据。

  7. 回调执行与流继续

    线程： 仍是 Event Loop 线程。

    动作： 读取的数据被传递给 R2DBC 驱动。R2DBC 驱动将数据格式化，并作为 onNext() 信号推送给上游的 Reactor 流。

    结果： 开发者定义的 map()、filter() 等所有回调操作开始被顺序执行。所有这些应用逻辑的执行，都发生在同一个 Event Loop 线程上，直到数据处理完毕，响应被发送回客户端。

  总结

    整个 R2DBC 过程的关键在于 第 4 步（控制权交出）。Event Loop 线程只负责发起请求和处理响应，它将中间的 I/O 等待时间完全释放出来，去服务成百上千的其他并发连接，从而保证了非阻塞和高吞吐量。

# 在I/O 注册和发送时， 注册的回调是什么内容？

  当前执行的环境（或者说“续点”）被保存了，但这个保存方式不是传统阻塞式线程挂起那样保存整个线程堆栈，而是以**回调链（Callback Chain）**的形式被保存。

  在 WebFlux/Reactor 模型中，我们不保存线程的状态，而是保存 数据返回后应该执行的逻辑。

 上下文是如何被“保存”的？

  当 Event Loop 线程在 I/O 注册后返回主循环时，以下两个关键部分充当了被保存的“执行上下文”：

    1. Reactor 层面：回调链和 Subscriber 对象

      在您的响应式代码（如 .map().filter().flatMap()...）被执行时，Reactor 框架做的工作是将这些操作符转换成了一个个相互链接的 Subscriber 对象。

      Subscriber 链即是续点： 当代码执行到 R2DBC 的 I/O 调用时，Mono 或 Flux 实际上是一个 Publisher，它持有一个或多个下游的 Subscriber（对应您的 .map()、.filter() 等操作）。

      保存的不是数据，而是逻辑： 整个 Subscriber 链就是被保存的执行计划——它定义了数据一旦就绪，要从哪个操作符开始继续执行。这个链条本身就是异步操作的上下文。

      闭包 (Closure) 捕获局部变量： 如果您的 map 或 filter 操作中使用了外部的局部变量，这些变量会被闭包（Closure）机制捕获，并作为 Subscriber 对象状态的一部分被保存下来。

    2. Netty 层面：I/O 句柄与回调的关联

      Netty/R2DBC 负责将这个执行计划（即回调链）与实际的 I/O 事件关联起来：

      I/O 句柄（Socket Channel）： R2DBC 驱动持有与数据库连接的 Socket 句柄。

      注册回调： 在 Event Loop 注册 I/O 兴趣到 Selector 时，它不仅注册了“当数据准备好时通知我”，还必须将 “数据准备好后要执行的回调函数（即 Reactor 的数据推送入口）” 关联到这个 Socket 句柄上。

      当内核通知 Event Loop 有数据就绪时，Event Loop 线程会：

      通过 Socket 句柄查找：根据内核报告的 Socket 句柄，找到与之关联的那个 回调函数/ Subscriber 链的入口。

      执行回调：Event Loop 线程调用该入口函数，开始将数据推入 Reactor 流中，继续执行被保存的 .map()、.filter() 等后续逻辑。

      总结：保存的不是线程，而是计划


    因此，当 Event Loop 交出控制权时：

    没有保存： 线程的整个堆栈、寄存器状态没有被保存。该线程立即返回，去处理其他连接。

    保存了： 数据库连接的 Socket 被注册到 Selector 上，并与一个回调对象（Reactor 链的入口） 关联起来。

    这个回调对象就是“被保存的执行环境”，它确保了当数据回来时，Event Loop 知道用哪个数据，从哪个位置，在哪个线程上（即它自己）继续执行。

# 当我们想自定义一个服务器，像netty，tomcat一样的服务器，我们该怎么做？

  核心改动：实现自定义 HttpHandler 适配器 。
