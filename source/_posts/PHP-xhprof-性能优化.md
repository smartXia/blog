---
title: PHP-xhprof-性能优化
date: 2020-08-20 17:16:14
tags: xhprof docker push
cover: /img/php.jpeg
categories:
  - PHP
---

### 简介

------



### 安装

------



### 嵌入代码

------



### 查看分析报告

------



### 总结
------

### 名词解释




Function Name：方法名称。

Calls：方法被调用的次数。

Calls%：方法调用次数在同级方法总数调用次数中所占的百分比。

Incl.Wall Time(microsec)：方法执行花费的时间，包括子方法的执行时间。（单位：微秒）

IWall%：方法执行花费的时间百分比。

Excl. Wall Time(microsec)：方法本身执行花费的时间，不包括子方法的执行时间。（单位：微秒）

EWall%：方法本身执行花费的时间百分比。

Incl. CPU(microsecs)：方法执行花费的CPU时间，包括子方法的执行时间。（单位：微秒）

ICpu%：方法执行花费的CPU时间百分比。

Excl. CPU(microsec)：方法本身执行花费的CPU时间，不包括子方法的执行时间。（单位：微秒）

ECPU%：方法本身执行花费的CPU时间百分比。

Incl.MemUse(bytes)：方法执行占用的内存，包括子方法执行占用的内存。（单位：字节）

IMemUse%：方法执行占用的内存百分比。

Excl.MemUse(bytes)：方法本身执行占用的内存，不包括子方法执行占用的内存。（单位：字节）

EMemUse%：方法本身执行占用的内存百分比。

Incl.PeakMemUse(bytes)：Incl.MemUse峰值。（单位：字节）

IPeakMemUse%：Incl.MemUse峰值百分比。

Excl.PeakMemUse(bytes)：Excl.MemUse峰值。单位：（字节）

EPeakMemUse%：Excl.MemUse峰值百分比。