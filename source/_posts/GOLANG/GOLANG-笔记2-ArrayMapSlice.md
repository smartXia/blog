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

#### 1.golang 数据类型

1、基本数据类型：整形、浮点、布尔、字符串、字符
2、复合数据类型：函数与指针、数组、切片、map、list、结构体、通道

#### 1.1 函数与指针 

函数：func（）
指针：* 代表指针 & 去地址符 （引用传值）

*可以表示一个变量是指针类型 , 也可以表示一个指针变量所指向的存储单元 , 也就是这个地址所存储的值 .
打印 *类型数据获取的一堆2进制数据

&对变量取地址 即取得某个变量的地址 , 如 ; &a


#### 1.2数组
Array(数组)
数组是很有价值的数据结构，因为它的内存分配是连续的，内存连续意味着可是让它在 CPU 缓存中待更久，所以迭代数组和移动元素都会非常迅速。
- 数组所有 的key 都是int 默认从0 开始，string类型的是map和list其他，不建议使用数组，一般直接上切片
- var 变量名 [数组长度]数据类型
```
	var a [4]int
	println(a)
	d := [3]int{1, 2, 4}
	println(d)
	f := [...]int{1, 2, 4, 5, 6} //... 省略数组长度 自动计算
	println(f)

	a2 := [...]int{1, 2, 4: 5, 6}
	println(a2)
	for key, val := range a2 {
		println(key,val)
	}
 // 二维数组赋值 
var arrayarr = [3][4]int{{1, 2, 3, 4}, {5, 6, 7, 8}, {9, 10, 11, 12}}
	dd := [...][...]string{
		{"3", "4"},
		{"c", "2"},
	}
	println(dd)
	println(arrayarr)
  
```
- 补充一个很恶心的代码
```
//补充一种确定下标的数组声明及定义方式：
func testArray04(){
	//a1数组下标0的没有定义，默认为0，下标1的是2，下标2的是3，
	a1 := [...]int{2:3,1:2} //这边的2 是key 打印按照key来打印
	fmt.Println(a1)      //[0 2 3]
 
	//a2数组下标4定义为5，之前下标2和3未定义，默认为0
	a2 := [...]int{1,2,4:5,6}
	fmt.Println(a2)      //[1 2 0 0 5 6]
}
```
#### 1.3 Map(集合) 
Map 是一种无序的键值对的集合。Map 最重要的一点是通过 key 来快速检索数据，key 类似于索引，指向数据的值。
Map 是一种集合，所以我们可以像迭代数组和切片那样迭代它。不过，Map 是无序的，我们无法决定它的返回顺序，这是因为 Map 是使用 hash 表来实现的。

声明： m := make(map[string]string)
```
ff := map[string]string{
		"a": "c",
	}
 var keys []string
  for k := range m {
    keys = append(keys, k) //append  直接在切片级别进行描述
  }
  
	for s, s2 := range ff {
		println(s,s2)
	}

ff := map[string]string{
		"a": "c",
}
	fff:=make(map[string]int) //直接声明不加参数
	ff["cc"]="cccc";//赋值
	delete(ff,"cc")
```

#### 1.4切片


#### 1.5 结构体
定义切片:
你可以声明一个未指定大小的数组来定义切片：
```
var identifier []type//未定义大小的数组即切片
```
或使用 make() 函数来创建切片:
```
slice1 := make([]type, len)  //make([]T, length, capacity)
```
append  和copy 浅拷贝
```
  var numbers []int
  numbers = append(numbers, 0)//append追加
    /* 同时添加多个元素 */
  numbers = append(numbers, 2,3,4)
   /* 拷贝 numbers 的内容到 numbers1 */
  copy(numbers1,numbers)
```


#### 1.6 List（链表）
遍历
```
// 声明链表
l := list.New()

// 数据添加到尾部
l.PushBack(4)
l.PushBack(5)
l.PushBack(6)

// 遍历
for e := l.Front(); e != nil; e = e.Next() {
     fmt.Printf("%v\n", e.Value)
}

 l := list.New()
 l.PushBack(4)
 six := l.PushBack(6)
 l.Remove(six) // 删除6这个节点
```
#### 1.6 通道
 其他文章会有详细讲述

#### 2总结
数组是 slice 和 map 的底层结构。
slice 是 Go 里面惯用的集合数据的方法，map 则是用来存储键值对。
内建函数 make 用来创建 slice 和 map，并且为它们指定长度和容量等等。slice 和 map 字面值也可以做同样的事。
slice 有容量的约束，不过可以通过内建函数 append 来增加元素。
map 没有容量一说，所以也没有任何增长限制。
内建函数 len 可以用来获得 slice 和 map 的长度。
内建函数 cap 只能作用在 slice 上。
可以通过组合方式来创建多维数组和 slice。map 的值可以是 slice 或者另一个 map。slice 不能作为 map 的键。
在函数之间传递 slice 和 map 是相当廉价的，因为他们不会传递底层数组的拷贝。