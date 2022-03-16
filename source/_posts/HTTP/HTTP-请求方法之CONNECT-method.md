---
title: HTTP 请求方法之CONNECT method
categories:
  - HTTP
date: 2021-11-25 09:36:06
tags:
cover: /img/cloud.jfif
---
> HTTP1.1 中的connect
1.http请求代理就是connect这个方法，connect网页开发中不会使用
2.connect的作用将服务器作为代理，让服务器提用户访问其他网页（翻墙），之后将数据返回用户
3.connect是将通过TCP代理链接服务器的，假如我想让代理服务器访问，https://baidu.com网站，首先要简历一条客户端到代理服务器的tcp的链接
然后给代理服务器发送一个http报文
```
CONNECT https://www.jianshu.com/u/f67233ce6c0c:80 HTTP/1.1
Host: www.web-tinker.com:80
Proxy-Connection: Keep-Alive
Proxy-Authorization: Basic *
Content-Length: 0
```
在发送完这个请求之后，代理服务器会响应请求，返回一个200的信息，但这个200并不同于我们平时见到的OK，而是Connection Established