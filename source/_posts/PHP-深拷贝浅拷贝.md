---
title: PHP-深拷贝浅拷贝-_clone
date: 2020-09-02 14:50:07
tags:
cover: img/php.jpeg
---
- 深拷贝//变量复制了一份传递给另一个变量就是深拷贝,一个值改变了,另一个值不会变（直接复制）


- 浅拷贝//变量之间的值是地址*|&传递,这就是浅拷贝.值如果改变了两个变量的值都会改变 （引用复制，可变）

> 关键次 _clone

- 对象赋值：浅拷贝
- 普通类型的变量是深拷贝

#### php默认浅拷贝即普通赋值

> 例1：
```
class Persion
{
public $age = 0;
public $name = 'xiapeifus';
public $obj = null;
}
$persion = new Persion();
$xiaoming = clone $persion; //使用clone关键字复制一份$a的值,进行深拷贝.拷贝之后不会改变$a之前的值
$xiaoming->age = 1;
var_dump($persion->age);// 0
var_dump($xiaoming->age);// 1
```

> //例2：增加__clone对象的赋值

```
class doclone{
    private $id,$name,$address;
    public function __construct($id=0,$name='',$address=''){
        $this->name=$name;
        $this->id=$id;
        $this->address=$address;
    }
    public function get_id(){
        return $this->id;
    }
    public function get_name(){
        return $this->name;
    }
    public function get_address(){
        return $this->address;
    }
    public function __clone(){
        $this->id=$this->id+1;
        $this->name='wqw';
        $this->address='USA';
    }
}

$A = new doclone(1,'xiapeifu','UK');
echo '克隆之前的对象:';
echo 'id='.$A->get_id();
echo 'name='.$A->get_name();
echo 'address='.$A->get_address();
echo "\n";


$B = clone $A;
echo '克隆过后的对象：';
echo 'id='.$A->get_id();
echo 'name='.$A->get_name();
echo 'address='.$A->get_address();
echo "\n";

echo '克隆过后的对象属性:';
echo 'id='.$B->get_id();
echo 'name='.$B->get_name();
echo 'address='.$B->get_address();
//结果
//克隆之前的对象:id=1name=xiapeifuaddress=UK
//克隆过后的对象：id=1name=xiapeifuaddress=UK
//克隆过后的对象属性:id=2name=wqwaddress=USA

```

> 思考：colne 关键词 当对象没__colne 方法时候类似于new 一个对象出来没区别
        当对象有__clone时候会在clone里面的重新赋值新的属性，类似于重新new 一个对象 ，只不过把重新new的对象进行一些默认操作，其实重新new一个对象重新赋值也一样，clone可能就是单纯炫技吧
        




