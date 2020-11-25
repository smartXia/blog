---
title: 前端-npm私有源加载平台
date: 2020-09-01 17:30:06
tags:
cover: /blog/img/npm.jpg
---

### 调研平台：sinopia，Verdaccio 

#### Verdaccio 

#####  官方文档

https://verdaccio.org/docs/en/configuration

搭建：

##### 认证方式

身份验证与您正在使用的auth [插件相关](https://verdaccio.org/docs/en/plugins)。软件包限制也由[Package Access](https://verdaccio.org/docs/en/packages)处理。

客户端：基于npm客户端登录后会生成一个配置令牌，在.npmrc中

https://docs.npmjs.com/files/npmrc

且允许匿名发布包

包发布的时候允许阻止访问和下载

服务端关于组的验证:

​    access: $all->
​    publish: $all
​    proxy: npmjs

![image-20200901175328057](//s2-cdn.oneitfarm.com/186bdf368bd54a57b12eb71ba2e10636.png)



 不同的包读取权限限制：

```js
packages:
  'jquery':
    access: $all
    publish: $all
  'my-company-*':
    access: $all
    publish: $authenticated
  '@my-local-scope/*':
    access: $all
    publish: $authenticated
  '**':
    access: $all
    publish: $authenticated
    proxy: npmjs
```

组 定义：

```
  'company-*':
    access: admin internal
    publish: admin
    proxy: server1
  'supersecret-*':
    access: secret super-secret-area ultra-secret-area
    publish: secret ultra-secret-area
    proxy: server1
```

