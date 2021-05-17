---
title: http与rpc区别，以及如何使用rpc
date: 2020-06-19 10:55:30
tags: rpc()
cover: /img/cloud.jfif
categories:
  - HTTP
keywords: 
 - protobuf

---
> HTTP就是一种RPC(Remote Procedure Call) ,http是七层iso模型，转换步骤多，且基于http有更多的报文

**http好比普通话，rpc好比团伙内部黑话。**
只要是远程调用都可以叫RPC()，和是不是通过http没什么关系
讲普通话，好处就是谁都听得懂，谁都会讲。
讲黑话，好处是可以更精简、更加保密、更加可定制，坏处就是要求“说”黑话的那一方（client端）也要懂，而且一旦大家都说一种黑话了，换黑话就困难了。

> 首先 http 和 rpc 并不是一个并行概念。

rpc是远端过程调用，其调用协议通常包含传输协议和序列化协议。

传输协议包含: 如著名的 [gRPC]([grpc / grpc.io](https://link.zhihu.com/?target=http%3A//www.grpc.io/)) 使用的 http2 协议，也有如dubbo一类的自定义报文的tcp协议。

序列化协议包含: 如基于文本编码的 xml json，也有二进制编码的 protobuf hessian等。

因此我理解的问题应该是：**为什么要使用自定义 tcp 协议的 rpc 做后端进程通信？**

解决这个问题就应该搞清楚 http 使用的 tcp 协议，和我们自定义的 tcp 协议在报文上的区别。

首先要否认一点 http 协议相较于自定义tcp报文协议，增加的开销在于**连接的建立与断开**。http协议是支持**连接池复用**的，也就是建立一定数量的连接不断开，并不会频繁的创建和销毁连接。要说的是http也可以使用protobuf这种二进制编码协议对内容进行编码，因此二者最大的区别还是在传输协议上。

通用定义的http1.1协议的tcp报文包含太多废信息，一个POST协议的格式大致如下：

```http
HTTP/1.0 200 OK 
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84

<html>
  <body>Hello World</body>
</html>
```

即使编码协议也就是body是使用二进制编码协议，报文元数据也就是**header头的键值对却用了文本编码，非常占字节数**。如上图所使用的报文中有效字节数仅仅占约 30%，**也就是70%的时间用于传输元数据废编码**。当然实际情况下报文内容可能会比这个长，但是报头所占的比例也是非常可观的。

那么假如我们使用自定义tcp协议的报文如下

![tcp](https://pic2.zhimg.com/80/v2-89c905b0806577471aa7789a25ac0d44_720w.jpg)

报头占用的字节数也就只有16个byte，极大地精简了传输内容。

这也就是为什么后端进程间通常会采用自定义tcp协议的rpc来进行通信的原因

简单来说成熟的rpc库相对http容器，更多的是封装了“服务发现”，"负载均衡"，“熔断降级”一类面向服务的高级特性。可以这么理解，rpc框架是面向服务的更高级的封装。如果把一个http servlet容器上封装一层服务发现和函数代理调用，那它就已经可以做一个rpc框架了。

> 那http和rpc和websocket三者有什么关系呢？

[Web Service](https://link.jianshu.com?t=http://en.wikipedia.org/wiki/Web_Service) 也提出了好久了, 那么究竟什么是 [Web Service](https://link.jianshu.com?t=http://en.wikipedia.org/wiki/Web_Service) ?
 简单地说, 也就是服务器如何向客户端提供服务.
 常用的方法有:
 [RPC](https://link.jianshu.com?t=http://en.wikipedia.org/wiki/Remote_procedure_call) 所谓的远程过程调用 (面向方法)
 [SOA](https://link.jianshu.com?t=http://en.wikipedia.org/wiki/Service-oriented_architecture) 所谓的面向服务的架构(面向消息)
 [REST](https://link.jianshu.com?t=http://en.wikipedia.org/wiki/Representational_State_Transfer) 所谓的 **Representational state transfer** (面向资源)



转载整理自：https://www.zhihu.com/question/41609070/answer/191965937