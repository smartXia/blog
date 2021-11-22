---
layout: golang3-笔记-array
title: GOLANG-笔记-ArrayMapSlice
date: 2021年4月26日
cover: /img/golang.jpeg
categories:
  - GOLANG
tags: 
  - Slice
keywords: Array Map Slice
---

```
### 1. := = ==

:= 给某变量的第一次赋值，初始化

= 变量的非第一次赋值

== 等于操作符

### 2. go中nil的使用

指针、切片、映射、通道、函数和接口的零值则是 nil。

- nil 标识符是不能比较的
- nil 不是关键字或保留字
- nil 没有默认类型
- 不同类型 nil 的指针是一样的
- 不同类型的 nil 是不能比较的
- 两个相同类型的 nil 值也可能无法比较
- nil 是 map、slice、pointer、channel、func、interface 的零值
- 不同类型的 nil 值占用的内存大小可能是不一样的

### 3.切片 slice

类似于py或Java的list ,是数组的抽象,支持数组扩容 定义:
slice1 := make([]type, len)

len 获取长度 cap 获取容量 append(slice1 ,v1,...) copy(new_slice,slice1)

### 4.集合 map

map 无序k-v ,快速根据k 找到v,类似于索引,在做循环打印的时候，无法固定返回顺序，因为map 用hash表来实现的

- 声明变量，默认 map 是 nil var map_val map[key_data_type]value_data_type

- 使用 make 函数 map_val := make(map[key_data_type]value_data_type)

### 并发
goroutine 
go sync()
### 通道 channel
既然已经有了线程的概念，那么就会存在线程间的同步和通讯问题，Go 使用通道（channel）来实现。

通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。使用操作符 <-，符号左边是接收者，右边是发送者。

使用 make 创建 channel，如下：

​```
ch := make(chan int, 100) // make 第二个参数 100 是该通道的缓冲区，是一个可选参数，如果不指定，那么就是无缓冲的通道
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据
// 并把值赋给 v
​```
通道与消息队列是等效的，如果通道缓冲区满，那么再往通道里塞数据，就会阻塞该 goroutine；同样，如果通道缓冲区没有数据了，再次接收通道数据，也会阻塞该 goroutine。
```