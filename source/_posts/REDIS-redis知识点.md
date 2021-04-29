---
title: redis进阶知识
date: 2019-12-27 11:01:49
tags: redis
keywords: "知识点"
cover: /img/redis.jpg
top_image: https://pic4.zhimg.com/v2-ca6f0a9731c98901ec1e3869a7a1df4e_1200x500.jpg
categories:
 - redis学习
---

### 关于redis 一些基本知识点

#### redis基本介绍

remote Dictionary Serer (远程字典服务) ANSI C 编写 支持网络 基于内存可持久化的日志型 key=>value 数据库

官网 redis.io   作者：Salvatore Sanfilippo 意大利人 网名 antirez

#### redis被使用的原因

  传统型关系型数据库如mysql 已经不能适用所有场景，秒杀库存 app首页流量访问高峰  所以考虑缓存中间件，目前市面上比较常用的就是redis 和memcached 

#### redis 基本的数据结构

String 、字典Hash、列表List、集合Set、有序集合 SortedSet、 HyperLogLog、Geo、Pub/Sub

##### BloomFilter 布隆过滤器

[避免缓存击穿的利器之BloomFilter](https://link.zhihu.com/?target=https%3A//juejin.im/post/5db69365518825645656c0de)

#### redis分布式锁

-先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放

#### redis关键的一个特性：

redis的单线程的。keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用`scan`指令，`scan`指令可以无阻塞的提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长

#### 使用Redis做异步队列

一般使用list结构作为队列，`rpush`生产消息，`lpop`消费消息。当lpop没有消息的时候，要适当sleep一会再重试。

list还有个指令叫`blpop`，在没有消息的时候，它会阻塞住直到消息到来。

#### 实现延时队列

使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用`zrangebyscore`指令获取N秒之前的数据轮询进行处理。

#### 生产一次消费多次

使用pub/sub主题订阅者模式，可以实现 1:N 的消息队列

缺点：消费者下线的话 生产者会消失，使用MQ

#### Redis是怎么持久化

RDB做镜像全量持久化，AOF做增量持久化。因为RDB会耗费较长时间，不够实时，在停机的时候会导致大量丢失数据，所以需要AOF来配合使用。在redis实例重启时，会使用RDB持久化文件重新构建内存，再使用AOF重放近期的操作指令来实现完整恢复重启之前的状态。

RDB的原理: fork和cow。fork是指redis通过创建子进程来进行RDB操作，cow指的是copy on write，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来。

```
这里很好理解，把RDB理解为一整个表全量的数据，AOF理解为每次操作的日志就好了，服务器重启的时候先把表的数据全部搞进去，但是他可能不完整，你再回放一下日志，数据不就完整了嘛。不过Redis本身的机制是 AOF持久化开启且存在AOF文件时，优先加载AOF文件；AOF关闭或者AOF文件不存在时，加载RDB文件；加载AOF/RDB文件城后，Redis启动成功； AOF/RDB文件存在错误时，Redis启动失败并打印错误信息
```

#### Pipeline有什么好处

可以将多次IO往返的时间缩减为一次，前提是pipeline执行的指令之间没有因果相关性。使用redis-benchmark进行压测的时候可以发现影响redis的QPS峰值的一个重要因素是pipeline批次指令的数目。

#### Redis的同步机制

Redis可以使用主从同步，从从同步。第一次同步时，主节点做一次bgsave，并同时将后续修改操作记录到内存buffer，待完成后将RDB文件全量同步到复制节点，复制节点接受完成后将RDB镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。后续的增量数据通过AOF日志同步即可，有点类似数据库的binlog。