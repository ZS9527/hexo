---
title: '使用rabbitmq时出现的Channel shutdown: channel error;'
date: 2019-10-26 16:15:48
tags: rabbiteMQ
---

# 使用rabbitmq时出现的Channel shutdown: channel error;
> 本次问题并不是需要很复杂的处理，但是还是要记录一下。

<!--more-->

## 错误：
**Channel shutdown: channel error; protocol method: #method<channel.close>**(reply-code=406, reply-text=PRECONDITION_FAILED - inequivalent arg 'x-message-ttl' for queue 'river.water.warning' in vhost ' /': received the value '60000' of type 'signedint' but current is none, class-id=50, method-id=10)

## 修复方法
这次的问题出现在我配置了**rabbitMQ**的**TTL**后出现的。原因是因为消费段和生产者的设置不同，会造成启动后的消息写入不到队列中。只需要把两边的配置同步即可解决