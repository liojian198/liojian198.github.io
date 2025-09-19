---
title: "alexa使用"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/alexa
date: 2025-09-20
---

# 概述

    在上班的时候，公司产品要接入alexa和home kit,发现有个大牛，在公司搞对接alexa 搞了1个月，连流程都搞不清楚，可能公司没有什么压力吧。不扯了，先上图，只有图，哈哈。

# alexa 对接全景交互时序图

<img src='/images/alexa/alexa.png'>

# alexa 用户系统与我们的用户系统打通

<img src='/images/alexa/tokens.png'>

    注意http code 的301使用。 这里不涉及APP里面弹起内置浏览器，然后app通浏览器里面拿code码的流程，不需要在APP端定义协议做这个事情。


# 大概的配置

 1. Alexa Developer Console。 后台设置，有话术，意图，lambda等的设置要注意。



