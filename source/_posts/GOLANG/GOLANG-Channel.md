---
title: golang调度-channel
cover: /img/golang.jpeg
categories:
  - golang
date: 2021-11-15 18:56:27
tags: 
  - golang
---
**Channel**

### 设计原理

不要通过恭喜内存的方式进行通信，二十通过通信的方式共享内存。

很多主流编程语言中，多个线程传递数据方式一般情况都是共享内存，为了解决线程竞争，需要限制同一时间读写这些变量的线程数量

虽然可以通过共享内存加互斥锁进行通信，但是go提供了一种不同并发的模型，即顺序通讯进程，

Gorouting 和channel分别对应csp中实体和传递信息媒介。

**gorutine通过channel传递数据**

两个独立的goroutine ，一个会向channel中发送数据，另一个会从channel中读取数据，两个能独立的运行，并不存在直接关联，但是通过channel完成通讯

**先入先出原则**（FIFO）

- 先从channel读取数据的goroutine会先接受到数据
- 先向channel发送数据的goroutine会得到先发送的权力



这种 FIFO 的设计是相对好理解的，但是稍早的 Go 语言实现却没有严格遵循这一语义，我们能在 [runtime: make sure blocked channels run operations in FIFO order](https://github.com/golang/go/issues/11506) 中找到关于带缓冲区的 Channel 在执行收发操作时没有遵循先进先出的讨论[2](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#fn:2)。



- 发送方会向缓冲区写入数据，然后唤醒接收方，多个接受方会先尝试从缓冲区读取数据，如果没有读取到会重新陷入休眠。
- 接收方会从缓冲区去读数据，然后唤醒接收方，发送方会尝试像缓冲去写入数据，如果缓冲区已满会重新陷入休眠

这种基于重试的机制会导致channel的处理遵循先进先出的原则。



**无锁管道**



**数据结构**

Go在channel中运行使用runtime.hchan ,新建chnanel结构

```go
type hchan struct {
	qcount   uint
	dataqsiz uint
	buf      unsafe.Pointer
	elemsize uint16
	closed   uint32
	elemtype *_type
	sendx    uint
	recvx    uint
	recvq    waitq
	sendq    waitq

	lock mutex
}
```



创建新的channel，如上结构构造地城循环队列：五个字段

- qcount channel中元素个数
- dataqsiz channel循环长度
- buf channel缓冲指针
- sendx channel发送操作处理到的位置
- recvx channel 接受的操作位置

除此之外，elemsize elemtype 标识channel收发的元素类型和大小

sendq和recvq存储当前channel由于缓冲区元素不足而阻塞的goroutine 列表，这些等待队列可以用双向列表runtime.waitq标识，链表中所有元素都是runtime.sudog

```go
type waitq struct {
	first *sudog
	last  *sudog
}
```

[`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 表示一个在等待列表中的 Goroutine，该结构中存储了两个分别指向前后 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 的指针以构成链表。



**创建管道**

go中所有channel节点创建都会使用make关键字，编译器会将make(chan int,10)表达式转换成OMAKE类型的节点，并在类型检查阶段，将OMAKEl类型节点转为OMAKECHAN类型：

```
func typecheck1(n *Node, top int) (res *Node) {
	switch n.Op {
	case OMAKE:
		...
		switch t.Etype {
		case TCHAN:
			l = nil
			if i < len(args) { // 带缓冲区的异步 Channel
				...
				n.Left = l
			} else { // 不带缓冲区的同步 Channel
				n.Left = nodintconst(0)
			}
			n.Op = OMAKECHAN
		}
	}
}
```

这一阶段会对传入 `make` 关键字的缓冲区大小进行检查，如果我们不向 `make` 传递表示缓冲区大小的参数，那么就会设置一个默认值 0，也就是当前的 Channel 不存在缓冲区。



- 如果当前 Channel 中不存在缓冲区，那么就只会为 [`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 分配一段内存空间；
- 如果当前 Channel 中存储的类型不是指针类型，会为当前的 Channel 和底层的数组分配一块连续的内存空间；
- 在默认情况下会单独为 [`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 和缓冲区分配内存；

在函数的最后会统一更新 [`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 的 `elemsize`、`elemtype` 和 `dataqsiz` 几个字段。



**发送数据**

当我们想要向Channel发送数据时候，就需要使用ch<-i语句，编译器将会将它解析冲OSEND节点，并在xxx ,转换runtime.channelsend1

```go
func walkexpr(n *Node, init *Nodes) *Node {
	switch n.Op {
	case OSEND:
		n1 := n.Right
		n1 = assignconv(n1, n.Left.Type.Elem(), "chan send")
		n1 = walkexpr(n1, init)
		n1 = nod(OADDR, n1, nil)
		n = mkcall1(chanfn("chansend1", 2, n.Left.Type), nil, init, n.Left, n1)
	}
}
```

[`runtime.chansend1`](https://draveness.me/golang/tree/runtime.chansend1) 只是调用了 [`runtime.chansend`](https://draveness.me/golang/tree/runtime.chansend) 并传入 Channel 和需要发送的数据。[`runtime.chansend`](https://draveness.me/golang/tree/runtime.chansend) 是向 Channel 中发送数据时一定会调用的函数，该函数包含了发送数据的全部逻辑，如果我们在调用时将 `block` 参数设置成 `true`，那么表示当前发送操作是阻塞的：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	lock(&c.lock)

	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}
```

在发送数据的逻辑执行之前会先为当前 Channel 加锁，防止多个线程并发修改数据。如果 Channel 已经关闭，那么向该 Channel 发送数据时会报 “send on closed channel” 错误并中止程序。

因为 [`runtime.chansend`](https://draveness.me/golang/tree/runtime.chansend) 函数的实现比较复杂，所以我们这里将该函数的执行过程分成以下的三个部分：

- 当存在等待的接收者时，通过 [`runtime.send`](https://draveness.me/golang/tree/runtime.send) 直接将数据发送给阻塞的接收者；
- 当缓冲区存在空余空间时，将发送的数据写入 Channel 的缓冲区；
- 当不存在缓冲区或者缓冲区已满时，等待其他 Goroutine 从 Channel 接收数据；



小结：

### 小结 [#](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#小结)

我们在这里可以简单梳理和总结一下使用 `ch <- i` 表达式向 Channel 发送数据时遇到的几种情况：

1. 如果当前 Channel 的 `recvq` 上存在已经被阻塞的 Goroutine，那么会直接将数据发送给当前 Goroutine 并将其设置成下一个运行的 Goroutine；
2. 如果 Channel 存在缓冲区并且其中还有空闲的容量，我们会直接将数据存储到缓冲区 `sendx` 所在的位置上；
3. 如果不满足上面的两种情况，会创建一个 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构并将其加入 Channel 的 `sendq` 队列中，当前 Goroutine 也会陷入阻塞等待其他的协程从 Channel 接收数据；

发送数据的过程中包含几个会触发 Goroutine 调度的时机：

1. 发送数据时发现 Channel 上存在等待接收数据的 Goroutine，立刻设置处理器的 `runnext` 属性，但是并不会立刻触发调度；
2. 发送数据时并没有找到接收方并且缓冲区已经满了，这时会将自己加入 Channel 的 `sendq` 队列并调用 [`runtime.goparkunlock`](https://draveness.me/golang/tree/runtime.goparkunlock) 触发 Goroutine 的调度让出处理器的使用权；