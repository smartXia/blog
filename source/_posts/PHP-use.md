---
title: PHP-use
date: 2020-06-23 18:04:39
cover: /blog/img/php.jpg
tags:
  - PHP
  - USE
keywords: trait  匿名函数 preg_replace_callback
categories:
  - PHP
---

php use用法
#### 1、user namespace
#### 2、use 一个trait 
针对于trait的即多继承

```php
trait A{ eat()} trait B{ drink()}  
class C {
	use A;
	use B;	
}

public Class D{
d=new Class C()
d->eat();
}
```

>  当不同的trait中，却有着同名的方法或属性，会产生冲突，可以使用insteadof或 as进行解决，insteadof 是进行替代，而as是给它取别名

```
use trait1,trait2{
        trait1::eat insteadof trait2;
        trait1::drive insteadof trait2;
        trait2::eat as eaten;
        trait2::drive as driven;
    }
```



#### 3.闭包->匿名函数

好处：节省内存  适合做回调函数

匿名函数：定义时未定义函数的名称
闭包： 创建时封装周围状态的函数，及时周围的环境不存在了，闭包中的状态还会存在

使用法则：