---
title: redis基础知识
date: 2019-12-27 11:48:42
tags: 
  - pubsub
keywords: "发布和订阅"
cover: /img/redis.jpg
top_image: https://pic4.zhimg.com/v2-ca6f0a9731c98901ec1e3869a7a1df4e_1200x500.jpg
categories:
  - redis
---
### redis pubsub 发布和订阅

常用命令：
> 查看发布的频道：PUBSUB CHANNELS

> 新建频道： PUBLISH mychannel "hello world"

> 订阅频道：SUBSCRIBE mychannel |   Psubscribe 多个channel |支持 channel* 格式

> 退订频道：UNSUBSCRIBE mychannel | Punsubscribe  多个channel  |支持 channel* 格式