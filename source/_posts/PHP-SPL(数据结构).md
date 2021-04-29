---
title: PHP-SPL(数据结构)
date: 2020-06-23 18:04:39
cover: /img/php.jpg
tags:
  - PHP
  - 数据结构
categories:
  - PHP
---
> php SPL四种常用的数据结构
1.栈【先进后出】
```
<span style="font-size:18px;">$stack = new SplStack();
$stack->push('data1');
$stack->push('data2');
$stack->push('data3');
echo $stack->pop();
 
//输出结果为
//data3</span><span style="font-size:24px;font-weight: bold;">
</span>
```

2.队列【先进先出 后进后出】


```
<span style="font-size:18px;">$queue = new SplQueue();
$queue->enqueue("data1");
$queue->enqueue("data2");
$queue->enqueue("data3");
echo $queue->dequeue();
//输出结果为
//data1</span>
```
3.堆
```
<span style="font-size:18px;">$heap = new SplMinHeap();
$heap->insert("data1");
$heap->insert("data2");
echo $heap->extract();
//输出结果为
//data1</span>
```

4.固定尺寸数组
```
<span style="font-size:18px;">$array = new SplFixedArray(5);
$array[0]=1;
$array[3]=3;
$array[2]=2;
var_dump($array);
//输出结果为
// object(SplFixedArray)[1]
// public 0 => int 1
// public 1 => null
// public 2 => int 2
// public 3 => int 3
```
————————————————
推荐学习：http://www.imooc.com/video/4849
原文链接：https://blog.csdn.net/zhengwish/article/details/51742264