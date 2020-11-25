---
title: PHP-匿名类匿名函数（闭包）
date: 2020-08-28 11:31:08
tags:
- Closure
- 匿名函数
- 匿名类
categories:
  - PHP
cover: /blog/img/php.jpg
---

#### 1.写一个匿名函数

> 类中 的写法

```php
//第一种写法
    public function qq()
    {
        $result = array_reduce([1, 2, 3, 4, 1], function ($result, $value) {
            return array_merge($result, array_values($value));
        }, array());
        return $result;
    }
//第二种写法
    public function qq2()
    {
        //将匿名函数交个一个变量
        $a = function ($result, $value) {
            return array_merge($result, array_values($value));
        };
        $result = array_reduce([1, 2, 3, 4, 1], $a, array());
        return $result;
    }
```

#### 2.理解一个闭包（匿名函数）

目前php用到闭包的数组函数包括：

```
array_map — 为数组的每个元素应用回调函数
array_walk — 使用用户自定义函数对数组中的每个元素做回调处理
array_reduce — 用回调函数迭代地将数组简化为单一的值
array_filter — 用回调函数过滤数组中的单元
该函数把输入数组中的每个键值传给回调函数。如果回调函数返回 true，则把输入数组中的当前键值返回结果数组中。数组键名保持不变。
array_intersect_uassoc — 带索引检查计算数组的交集，用回调函数比较索引
array_intersect_ukey — 用回调函数比较键名来计算数组的交集
array_reduce — 用回调函数迭代地将数组简化为单一的值
拼接成类似 (1,2,3,4,5) 

array_walk_recursive — 对数组中的每个成员递归地应用用户函数
----等等
常用的就是: array_map array_walk

```

#### 3.临时总结

**异同点**
 array_filter() 重点在于过滤（而不是新增）某个元素，当你处理到一个元素时，返回过滤后的数组
 array_map() 重点在于遍历一个数组或多个数组的元素，返回一个新的数组
 array_walk() 重点在于遍历数组进行某种操作

 array_filter() 和 array_walk()对一个数组进行操作，数组参数在前，函数参数在后
 array_map() 可以处理多个数组，因此函数参数在前，数组参数在后，可以根据实际情况放入多个数组参数