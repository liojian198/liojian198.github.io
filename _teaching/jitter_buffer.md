---
title: "jitter buffer 详解"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/jitter_buffer
date: 2026-04-30 16:19:00
---

# 简介

    最近在做ai陪伴语音聊天时，用到了VoIP相关技术，在llm生成内容，转换成语音时（TTS）， 需要通过wss/udp 发送到设备端ESP32—P4，设备对音频的处理也是很“简陋”， 拿到opus编码数据，就丢给解码器进行处理，再进行播放。RTP我们用的是websocket，
  通过传递opus的二进制数据到设备，我们是在服务器端进行了 jitterbuffer 处理， 和webrtc的实现不一样， 在设备端没有处理jitterbuffer的相关内容，无脑的拿到opus流，进行解码，播放。没有jitterbuffer的任何操作。 服务端就通过burst模型，
  绝对时间锚点发送。通过这种手段来控制音频流下发的节奏，这样方便及时打断，各种操作都在服务器端。这也让我们遇到了不少问题，VoIP技术相关问题描述见(https://liojian198.github.io//talks/VoIP_problem). 我们目前的服务是java写的，哈哈，
  懂的都懂，在RT系统领域中，哈哈，哈哈，哈哈， 要是真的对这块有性能要求， 到时候切到webrtc 那套里面了， 各种框架，各种服务，各种开源，各种组件都可以用，现在就苦逼了。

# 参考资料

  https://www.asterisk.org/jitter-buffer-asterisk/

  https://juejin.cn/post/6940089874317836302

  https://developer.volcengine.com/articles/7317915600867557430

  https://zhuanlan.zhihu.com/p/336489810
