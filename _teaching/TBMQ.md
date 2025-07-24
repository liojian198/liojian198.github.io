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

