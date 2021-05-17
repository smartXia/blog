---
layout: golang1-笔记2-array
title: GOLANG-笔记-ArrayMapSlice
date: 2020-06-29 19:17:32
cover: /img/golang.jpeg
categories:
  - golang
tags: 
  - Slice
keywords: Array Map Slice
---

Array(数组)
内部机制
在 Go 语言中数组是固定长度的数据类型，它包含相同类型的连续的元素，这些元素可以是内建类型，像数字和字符串，也可以是结构类型，元素可以通过唯一的索引值访问，从 0 开始。

数组是很有价值的数据结构，因为它的内存分配是连续的，内存连续意味着可是让它在 CPU 缓存中待更久，所以迭代数组和移动元素都会非常迅速。



总结
数组是 slice 和 map 的底层结构。
slice 是 Go 里面惯用的集合数据的方法，map 则是用来存储键值对。
内建函数 make 用来创建 slice 和 map，并且为它们指定长度和容量等等。slice 和 map 字面值也可以做同样的事。
slice 有容量的约束，不过可以通过内建函数 append 来增加元素。
map 没有容量一说，所以也没有任何增长限制。
内建函数 len 可以用来获得 slice 和 map 的长度。
内建函数 cap 只能作用在 slice 上。
可以通过组合方式来创建多维数组和 slice。map 的值可以是 slice 或者另一个 map。slice 不能作为 map 的键。
在函数之间传递 slice 和 map 是相当廉价的，因为他们不会传递底层数组的拷贝。