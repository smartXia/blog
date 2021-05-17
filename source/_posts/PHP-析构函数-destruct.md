---
title: PHP-析构函数-destruct
date: 2021-05-10 10:54:39d
tags: php
cover: img/php.jpeg
---
> phpStrom 里alt+insert 会出现的一些函数
###  析构函数destruct

![image.png](http://s2-cdn.oneitfarm.com/d7f45961508248afb2f08e0bd137ca9c.png)

简单理解：构造函数的对立面
构造函数：__construct()在初始化对象的时候默认执行的
析构函数：__destruct()在对象销毁回收时候默认执行的，类似于web框架里面的钩子函数

触发条件 当对象或者变量 消失时候

关键词：unset或者对象生命周期结束

phpStrom 里alt+insert 会出现的一些函数

```
calss A{
    
protected $data = [];

public function insert($data)
    {
        $data['appkey'] = getAppkey();
        $data['channel'] = getChannel();
        $this->data[] = $data;
        //这个[]意思在多个多次调用的时候插入整个数组很关键，可以看下面内容 请求中 php 如何分配phpfpm
    }
 public function __destruct()
    {
        if ($this->data) {
            $this->getDB()->insert_batch($this->table, $this->data);
            $id = $this->getDB()->insert_id();
            Ioc()->CallRecordModel->_delete([
                'id <' => $id - 50000
            ], '', 1000);
        }
    }
}

$zend=new A();
$zend->insert(["aaaaa"]);

```