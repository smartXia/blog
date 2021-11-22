---
title: GOLANG-纯纯的语法仔的没落
categories:
  - GOLANG
date: 2021-11-17 11:57:39
tags:
  - golang
cover: /img/golang.jpeg
---
## 基础

### 1. 对于已经关闭的channel的处理

读已经关闭的channel，一直能读到东西，但是读到的东西根据通道内关闭前是否有元素而不同，

- 关闭前，buffer有还未读取的，会读取到chan内的值，且返回是否读取成功的bool值为true
- 关闭前，buffer的值读完，channel元素值为0，bool值为false

写入：直接panic

### 2. Make和new区别

make：返回特定类型channel，slice,map

new: 开辟新内存和指针，泛化类型

### 3. nil 切片和空切片一不一样

指向的地址不一样。nill引用指针地址为0，空切片执行数组指针地址，且为一个固定值

数据结构：data,len,cap

### 4. 字符串转byte数组，会发生内存拷贝吗

严格来说，只要发生类型强转，都会发生内存拷贝。

那么go有个很强的包叫 `unsafe` 。先获取变量地址，字符串转成底层结构，通过unsafe包，转为切片数组,再通过指针指向实际内容

string 数据结构 {data,len} 

### 5. json包变量不加tag会怎么样

和key的大小写有关



### 6. GPM

指向另一篇详细（）

### 7.Docker 的网络通信模式。

四种：

1.host模式：和宿主机公用一个network NameSpace 。容器不会配置任何自己网卡，而是使用自己宿主机的IP和端口

2.container模式：指定和其他容器共享network nameSpace,而不是和宿主机共享

3.none模式：告诉容器放到自己网站堆里，但是不要配置他的网络

4.brideg模式：docker默认的网络模式，此模式会将主机docker链接到虚拟网桥上

### 8.访问私有成员

> 调用其他包共有结构的私有成员变**量**

**绕过小写不公开**

用unsafe包中的unsafe.Pointer获取到结构体对象的首地址，然后加上想访问的私有变量的偏移地址就是私有变量的地址



### 9、数组和切片的区别

长度，容量，数组指针

切片是指针类型，数组是是值类型

数据长度固定，切片不固定

切片比数组多个属性（cap）,切片底层是数组

- 扩容：小于1024 每次cap翻倍，超过变成1.25
- 扩容后没触及原数组容量，那么切片指针指向的位置，还是原数组，扩容后，超过原数组容量，会开辟一块新内存，原来的值拷贝过来，也不会影响原来数组
- append:

### 10.介绍 rune 类型

```

package main
import "fmt"
func main() {
    var str = "hello 你好"
    fmt.Println("len(str):", len(str))
    //12个 
    //中文字符在unicode下占2个字节，在utf-8编码下占3个字节 go默认utf-8 5+1+3*2
    //通过rune类型处理unicode字符
    fmt.Println("rune:", len([]rune(str))) //8个
    fmt.Println("RuneCountInString:", utf8.RuneCountInString(str))
}
```

byte等同于uint8，而不是int8

rune 等同于int32,常用来处理unicode或utf-8字符

### 11 panic defer recover

panic() 函数

函数中遇到panic语句，会立即终止当前函数的执行，在panic所在函数内如果存在要执行的defer函数列表，按照defer的逆序执行

recover() 函数

recover函数的返回值报告协程是否正在遭遇panic

有异常时，recover()只能调用一次，后面再次调用则捕获不到任何异常

通常办法：go中可以抛出一个panic的异常，然后在defer中通过recover捕获这个异常，然后正常处理，从而恢复正常代码的执行


### 12 读写锁和互斥锁

总结：
1. 1.在单纯的只是获取锁和释放锁时，互斥锁的用时要少一些，这主要是因为多个线程同时获取读写锁的情况比较少出现。
2. golang底层实现上，互斥锁确实要比读写锁的性能要好一些，这主要是因为读写锁的底层实现其实是互斥锁加上计数器
3. 在 增 强 协 程 互 相 冲 突 的 效 果 后 ， 读 写 锁 的 性 能 要 明 显 高 于 互 斥 锁 

### 13.结构体是否可以比较

回到上面的划重点部分，在总结中我们可以知道，golang中 *Slice*，*Map*，*Function* 这三种数据类型是不可以直接比较的。我们再看看S结构体，该结构体并没有包含不可比较的成员变量，所以该结构体是可以直接比较的。

 *reflect.DeepEqual 函数* 来对两个变量进行比较。

### 14.golang channel是线程安全的吗

如果把线程安全定义为允许多个goroutine同时去读写，那么golang 的channel 是线程安全的。不需要在并发读写同一个channe时加锁。

### 15.channel数据结构

```go
type hchan struct {
    qcount   uint           // total data in the queue
    dataqsiz uint           // size of the circular queue
    buf      unsafe.Pointer // points to an array of dataqsiz elements
    elemsize uint16
    closed   uint32
    elemtype *_type // element type
    sendx    uint   // send index
    recvx    uint   // receive index
    recvq    waitq  // list of recv waiters
    sendq    waitq  // list of send waiters

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex
}

type waitq struct {
    first *sudog
    last  *sudog
}
```
