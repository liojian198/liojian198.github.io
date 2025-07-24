---
title: "TBMQ"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/TBMQ
date: 2025-07-25
---

# 为什么要聊TBMQ

  今天面试的时候。问贵公司使用什么MQTT框架。他们基于thingsboard 二次开发的。mqtt是使用了thingboard的TBMQ,这个框架以之前也听过，但是没有去细看。mqtt相关的东西，我的关注点是在EMQX, MOP(MQTT OF PULSAR),MOK(MQTT OF KAFKA),MOR(MQTT OF ROCKETMQ)及云厂商的iot core， 例如: aws iot core, aliyun iot core,腾讯云 iot core 等。

  回来的时候，打开了TBMQ的官网看了下，找到了对应的架构设计。发现TBMQ的设计理念和我之前在JM的时候设计网关的理念是一样的。但是但是我在JM的时候， 用的是基于TCP之上自定义的协议，没有使用MQTT协议。但是考虑的是KAFKA topic过多的时候，会出现瓶颈（为什么会有这计划，看下面的详解）。当时JM 是有近2000万设备的，实时在线是有400W+的设备的。所以当时还是走了netty+ignite，来做了优化。将设备的连接和解包分开了处理，netty只做连接和解决半包，粘包，及指令下发的连接查询等简单，不用经常启动的工作。

## TBMQ 的C4模型图(自己画的)

   查看TBMQ代码几个小时，凭着理解画了个C4模型图，要是有问题后续更改.

### TBMQ 系统图

<img src='/images/tbmq/TBMQ_C4_system.png'>

### TBMQ 容器图

<img src='/images/tbmq/TBMQ_C4_container.png'>

### TBMQ 组件图

<img src='/images/tbmq/TBMQ_C4_compoent.png'>


# 进入TBMQ世界

## 开发TBMQ的动机

  我们识别出基于 MQTT 的解决方案的三个主要场景，
  
  1. 在第一个场景中，大量设备生成大量消息，这些消息被特定应用程序消费，形成扇入模式。通常，会设置几个应用程序来处理这些大量传入的数据。必须确保它们不会错过任何一条消息。

  2. 第二个场景涉及大量设备订阅特定的更新或通知，这些通知必须被传递。这导致少量传入请求引发大量传出数据。这种情况被称为扇出（广播）模式。
  
  3. 第三种场景，点对点（P2P）通信，是一种定向消息模式，主要用于一对一通信。适用于私信或基于命令的交互等用例，其中消息通过唯一定义的主题在单个发布者和特定订阅者之间进行路由。

  考虑到这些场景，我们有意设计 TBMQ 使其特别适用于所有三种情况。

  1. 我们的设计原则专注于确保代理的容错性和高可用性。因此，我们有意避免依赖主节点或协调进程。我们确保集群内所有节点都具有相同的功能。

  2. 我们优先支持分布式处理，允许随着业务的增长轻松实现水平扩展。我们希望我们的代理能够支持高吞吐量，并保证向客户端传递消息的低延迟。确保数据的持久性和复制在我们的设计中至关重要。我们旨在构建一个系统，一旦代理确认收到消息，该消息就会安全存储且不会丢失。

  3. 为确保满足上述要求，并防止在客户端或部分代理实例出现故障时消息丢失，TBMQ 利用 Kafka 强大的功能作为其底层基础设施。

## TBMQ 是如何工作的

1. Kafka 在 MQTT 消息处理的各个阶段都起着至关重要的作用。所有未处理的发布消息、客户端会话和订阅信息都存储在专门的 Kafka 主题中。TBMQ 中使用的 Kafka 主题的完整列表可以在后续部分找到。所有代理节点都可以通过使用这些主题来轻松访问客户端会话和订阅的最新状态。它们维护会话和订阅的本地副本，以实现高效的消息处理和投递。当客户端与特定代理节点失去连接时，其他节点可以基于最新状态无缝继续操作。此外，新加入集群的代理节点在激活时也会获取这些关键信息。

2. 客户端订阅在 MQTT 发布/订阅模式中具有重要意义。TBMQ 采用 Trie 数据结构来优化性能，能够高效地在内存中持久化客户端订阅，并促进对相关主题模式的快速访问。

3. 当发布者客户端发送 PUBLISH 消息时，该消息会存储在初始 Kafka 主题 tbmq.msg.all 中。一旦 Kafka 确认消息的持久化，代理会根据所选的 QoS 级别立即向发布者响应 PUBACK/PUBREC 消息或完全不响应。

4. 随后，作为 Kafka 消费者的独立线程会从提到的 Kafka 主题中消费消息，并利用订阅 Trie 数据结构来识别目标接收者。根据客户端类型（DEVICE 或 APPLICATION）以及下文描述的持久化选项，代理会将消息重定向到另一个特定的 Kafka 主题或直接交付给接收者。


## Kafka topics 及 kafka 对应的 topic 消费线程有哪些及执行什么样的逻辑

以下是 TBMQ 中使用的 Kafka 主题的全面列表及其相应描述。

### tbmq.msg.all - 用于存储从 MQTT 客户端发布到代理的所有消息的主题。

#### 描述

  这是所有 MQTT 发布消息的原始入口点。

#### 消费者 

TBMQ 集群中的所有 Broker 节点。每个 Broker 节点都会消费这个主题，以便：
识别并转发消息到其本地连接的订阅者。

将消息（如果需要）转发到其他 Kafka 主题（如 tbmq.msg.app.*、tbmq.msg.persisted、tbmq.msg.retained）进行进一步处理和路由。
计算全局统计数据。

#### 消费者类型

  内部消息路由器/分发器线程或服务。

## tbmq.msg.app. + ${client_id} - 用于存储基于其订阅应用程序客户端应接收的消息的主题。



tbmq.msg.app.shared. + ${topic_filter} - 主题用于存储应用程序客户端应接收的消息，这些消息基于它们共同的共享订阅。

tbmq.msg.persisted - 主题用于存储设备持久客户端应接收的消息，这些消息基于它们的订阅。

tbmq.msg.retained - 用于存储所有持久消息的主题。与 MQTT 持久消息功能相关

tbmq.client.session - 用于存储所有客户端会话的主题。

tbmq.client.subscriptions - 用于存储所有客户端订阅的主题。

tbmq.client.session.event.request - 用于存储所有客户端会话的事件，如 CONNECTION_REQUEST、DISCONNECTION_REQUEST、CLEAR_SESSION_REQUEST 等的主题。

tbmq.client.session.event.response. + ${service_id} - 主题用于存储针对先前主题发送到特定代理节点的响应事件，其中目标客户端已连接。

tbmq.client.disconnect. + ${service_id} - 主题用于存储强制客户端断开连接事件（由 UI/API 管理员请求或会话冲突触发）。

tbmq.msg.downlink.basic. + ${service_id} - 主题用于将消息从一个代理节点发送到当前已连接的 DEVICE 订阅者所在的另一个代理节点。

tbmq.msg.downlink.persisted. + ${service_id} - 主题用于将消息从一个代理节点发送到当前已连接的 DEVICE 持久订阅者所在的另一个代理节点。

tbmq.sys.app.removed - 用于处理 APPLICATION 客户端主题删除事件的发布主题。当客户端类型从 APPLICATION 更改为 DEVICE 时使用。

tbmq.sys.historical.data - 用于发布每个集群中每个代理节点的历史数据统计信息（例如，接收消息数量、发送消息数量等）的主题，以计算每个集群的总值。



## 持久化相关内容

  有用TBMQ的设计是代理是无状态的，理论上库水平扩点，但是state 部分的瓶颈也是影响到代理的扩容。所以扩容不是看单一的内容的扩容，要考虑上下游。TBMQ是使用了中心化的状态持久化， 不像EMQX使用去中心化的方式，在每个node下面有个sqlite，使用EMQX的数据一致性的局限性比较大，但是没有单点问题，比较灵活，自治。 根据文档信息，TBMQ的持久化分为设备持久化和应用的持久化。

  ### 设备持久化

  #### device_session_ctx 表格负责维护每个持久 MQTT 客户端的会话状态.

  Table "public.device_session_ctx"
       Column       |          Type          | Collation | Nullable | Default 
--------------------+------------------------+-----------+----------+---------
 client_id          | character varying(255) |           | not null | 
 last_updated_time  | bigint                 |           | not null | 
 last_serial_number | bigint                 |           |          | 
 last_packet_id     | integer                |           |          | 
Indexes:
    "device_session_ctx_pkey" PRIMARY KEY, btree (client_id)


关键列是 last_packet_id 和 last_serial_number ，用于维护持久 MQTT 客户端的消息顺序：

last_packet_id 代表最后接收到的 MQTT 消息的包 ID。

last_serial_number 作为持续递增的计数器，在 MQTT 包 ID 达到 65535 的限制后回绕时，防止消息顺序问题。


#### device_publish_msg 表格负责存储必须发布给持久 MQTT 客户端（订阅者）的消息。

                        Table "public.device_publish_msg"
          Column          |          Type          | Collation | Nullable | Default 
--------------------------+------------------------+-----------+----------+---------
 client_id                | character varying(255) |           | not null | 
 serial_number            | bigint                 |           | not null | 
 topic                    | character varying      |           | not null | 
 time                     | bigint                 |           | not null | 
 packet_id                | integer                |           |          | 
 packet_type              | character varying(255) |           |          | 
 qos                      | integer                |           | not null | 
 payload                  | bytea                  |           | not null | 
 user_properties          | character varying      |           |          | 
 retain                   | boolean                |           |          | 
 msg_expiry_interval      | integer                |           |          | 
 payload_format_indicator | integer                |           |          | 
 content_type             | character varying(255) |           |          | 
 response_topic           | character varying(255) |           |          | 
 correlation_data         | bytea                  |           |          | 
Indexes:
    "device_publish_msg_pkey" PRIMARY KEY, btree (client_id, serial_number)
    "idx_device_publish_msg_packet_id" btree (client_id, packet_id)


注意: TBMQ官网使用redis来优化性能。