---
title: GOLANG笔记1-基础数据类型
date: 2020-06-09 17:36:01
tags:
cover: /img/golang.png
categories:
  - golang
---
### int  和 uint
uint类型长度取决于 CPU，如果是32位CPU就是4个字节，如果是64位就是8个字节。我的电脑是64位的，而 playground 是32位的


> int是带符号的，表示范围是：-2147483648到2147483648，即-2^31到2^31次方。

> uint则是不带符号的，表示范围是：2^32即0到4294967295。

> uint可以使用十进制，二进制，十六进制。和long,ulong,float,double,decimal等预定义可以进行隐式转换。但是需要注意值是否在可转换的范围内，不然会出现异常。

The Uint keyword signifies an integral type that stores calues according to the size and ranges shown in the following table.

> 关键字表示一种整型类型，该类型根据下表显示的大小和范围存储值。
————————————————

原文链接：https://blog.csdn.net/janny_flower/article/details/81082424