---
title: GOLANG-GPM的深入理解
categories:
  - GOLANG
date: 2021-11-18 10:08:03
tags:
  - golang
cover: /img/golang.jpeg
---
深入golang runtime的调度

### 理解调度器的启动

runtime：

scheduler:

TLS:

spinning:

systemstack,mcall,asmcgocall

主要源码文件:

调度基本组件：

**G(goroutine)**：调度器的基本单位，存储的goroutine的执行stack信息，状态以及任务函数

在g的眼中只有p,p就是运行的G的“CPU”

相当于两级线程

> g的任务函数

每个g的实例都有任务函数，如下代码

```
userFun:=func(){fmt.Println("111")}
go userFunc();
```

go的关键词创建了一个goroutine,此时gouroutine的任务函数userFun

**P（processor）**

p表示逻辑processor，代表M执行的上下文

p的最大作用是拥有各种G的对象队列，链表，cache,和状态

p的数量也代表go的执行并发度，即多少个goroutine可以同时执行

这里的p虽然表示逻辑处理器，但是p并不代表任何执行代码，对于g来说，p相当于cpu的核，g只有绑定p才能调度。对于M来说，p提供了执行环境（Context），如分配内存状态（mcache）,任务队列G等

**M(machine)**

M代表真正的执行计算资源，可以任务他就是os thread(系统线程)

M是真正的执行者，每个M就像一个勤劳的工作者，总是从各种队列找到可运行的G,而且这样的M的可以同时存在多个

M在绑定有效P，可以进行调度循环，而且M并不保留G状态，这个是g可以跨M调度的基础













