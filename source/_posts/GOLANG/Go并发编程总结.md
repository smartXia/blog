---
title: Go并发编程总结
date: 2020-06-09 17:36:01
tags:
cover: /img/golang.jpeg
categories:
  - GOLANG
---


Golang :不要通过共享内存来通信，而应该***\*通过通信来共享内存\****。这句风靡在Go社区的话,说的就是 goroutine中的 channel。他在go并发编程中充当着类型安全的管道作用。

# 1、通过golang中的 goroutine 与sync.Mutex进行 并发同步

```csharp
package main

import(
	"fmt"
	"sync"
	"runtime"
)
var count int = 0   //全局共享变量

func counter(lock * sync.Mutex){  //goroutinue执行函数
	lock.Lock()
	count++
	fmt.Println(count)
	lock.Unlock()
}

func main(){
	lock := &sync.Mutex{}
	
	for i := 0; i < 10; i++ {
		//传递指针是为了防止 函数内的锁和 调用锁不一致    创建10个协程对count++
		go counter(lock)
	}
	for {
		lock.Lock()
		c := count
		lock.Unlock()
		///把时间片给别的goroutine  未来某个时刻运行该routine
		runtime.Gosched()
		if c >=10 {
			fmt.Println("goroutine end")
			break
		}
	}
}  //输出结果：1 2 3 4 5 6 7 8 9 10    "goroutine end"
```

# 2、goroutine之间通过 channel进行通信,channel是和类型相关的 可以理解为  是一种类型安全的管道。

```csharp
package main

import "fmt"

func Count(ch chan int) {
	ch <- 1
	fmt.Println("Counting")
}
func main() {
	chs := make([]<strong>chan int</strong>, 10)   //[]chan int 表示是chan int类型的切片 和   chan []int区别  标识通道类型是[]int
	//c := make(chan []int)
	//t := []int{1,2,3}
	//c <- t
	for i := 0; i < 10; i++ {
		chs[i] = make(chan int)
		go Count(chs[i])         //创建10个协程
		fmt.Println("Count",i)
	}
	for i, ch := range chs {
		<-ch
		fmt.Println("Counting",i)
	}
}       //调度的顺序不同，可能goroutinue执行的顺序不一样
```

# 3、Go语言中的select是语言级内置  非堵塞

```csharp
select {
case <-chan1: // 如果chan1成功读到数据，则进行该case处理语句  
case chan2 <- 1: // 如果成功向chan2写入数据，则进行该case处理语句  
default: // 如果上面都没有成功，则进入default处理流程  
}
```

可以看出，select不像switch，后面并不带判断条件，而是直接去查看case语句。每个case语句都必须是一个面向channel的操作。比如上面的例子中，第一个case试图从chan1读取一个数据并直接忽略读到的数据，而第二个case则是试图向chan2中写入一个整型数1，如果这两者都没有成功，则到达default语句。

# 4、channel 的带缓冲读取写入



要创建一个带缓冲的channel，其实也非常容易：



```go
c := make(chan int, 1024)
```

在调用make()时将缓冲区大小作为第二个参数传入即可，比如上面这个例子就创建了一个大小为1024的int类型channel，即使没有读取方，写入方也可以一直往channel里写入，在缓冲区被填完之前都不会阻塞。

从带缓冲的channel中读取数据可以使用与常规非缓冲channel完全一致的方法，但我们也可以使用range关键来实现更为简便的循环读取：

```go
for i := range c {
    fmt.Println("Received:", i)
} 
```

# 5、用goroutine模拟生产消费者

因为程序会***\*优先执行主线程，主线程执行完成后，程序会立即退出\****，没有多余的时间去执行子线程。如果在程序的最后让主线程休眠1秒钟，那程序就会有足够的时间去执行子线程。 通道又叫channel，顾名思义，channel的作用就是在多线程之间传递数据的。



```go
chreadandwrite :=make(chan int)
chonlyread := make(<-chan int) //创建只读channel    var read <-chan int = chreadandwrite 
chonlywrite := make(chan<- int) //创建只写channel   var write chan<- int = chreadandwrite 
package main

import (
	"fmt"
	"time"
)

func Producer(p chan<- int) {
	for i := 0; i < 10; i++ {
		p <- i
		fmt.Println("send:", i)
	}
}
func Consumer(c <-chan int) {
	for i := 0; i < 10; i++ {
		v := <-c
		fmt.Println("receive:", v)
	}
}

func main(){
	queue := make(chan int)
	/*ch :=make(chan int)
	ch <- 1
	这个错误的意思是说线程陷入了死锁，程序无法继续往下执行。那么造成这种错误的原因是什么呢？


<span style="white-space:pre;">	</span>我们创建了一个无缓冲的channel，然后给这个channel赋值了，程序就是在赋值完成后陷入了死锁。
<span style="white-space:pre;">	</span>因为我们的channel是无缓冲的，即同步的，赋值完成后来不及读取channel，程序就已经阻塞了。  =====    顺序执行满阻塞
<span style="white-space:pre;">	</span>queue <- 1
<span style="white-space:pre;">	</span>queue <- 2   //满阻塞
	*/

	go Producer(queue)     //开启协程执行函数单独去执行，主程序直接往下执行
	                       //i = 0 的时候，写入阻塞      生产者和消费者是异步执行，消费者会去读取channel值。
	go Consumer(queue)
	time.Sleep(1e9)        //让Producer与Consumer完成
}
```

因为channel是没有缓冲的，所以当生产者给channel赋值后，生产者这个线程会阻塞，***\*直到消费者线程将channel中的数据取出\****。消费者第一次将数据取出后，进行下一次循环时，消费者的线程也会阻塞，因为生产者还没有将数据存入，这时程序会去执行生产者的线程。程序就这样在消费者和生产者两个线程间不断切换，直到循环结束。

```go
package main

import (
	"fmt"
	"time"
)

func produce(p chan<- int) {
	for i := 0; i < 10; i++ {
		p <- i
		fmt.Println("send:", i)
	}
}
func consumer(c <-chan int) {
	for i := 0; i < 10; i++ {
		v := <-c
		fmt.Println("receive:", v)
	}
}
func main() {
	ch := make(chan int, 10)  //带缓冲的例子
	go produce(ch)
	go consumer(ch)
	time.Sleep(1 * time.Second)
}
```

在这个程序中，缓冲区可以存储10个int类型的整数，在执行生产者线程的时候，线程就不会阻塞，一次性将10个整数存入channel，在读取的时候，也是一次性读取。读取的话，没有数据就阻塞，有的话依次进行读取操作。

# 6、通过make 创建通道

```go
make(c1 chan int)       创建的是 同步channel ...读写完全对应    先写满的话就会阻塞  顺序程序不会执行
make(c1 chan int ,10)   创建带缓冲的通道 上来可以写10次        
```

# 7、随机向通道中写入0或者1 

```go
package main

import (
	"fmt"
	"time"
)

func main(){
	ch := make(chan int, 1)

	for i := 0; i < 10; i++ {
		///不停向channel中随机写入 0 或者1
		select {
		case ch <- 0:
		case ch <- 1:
		}
		//从通道中取出数据
		i := <-ch
		fmt.Println("Value received:",i)
		time.Sleep(1e8)
	}
}
```

# 8、带缓冲的channel 

之前创建的都是不带缓冲的channel，这种做法对于传递单个数据的场景可以接受，但对于需要持续传输大量数据的场景就有些不合适了。接下来我们介绍如何给channel带上缓冲，从而达到消息队列的效果。

要创建一个带缓冲的channel，其实也非常容易：



```go
c := make(chan int, 1024)
```

在调用make()时将缓冲区大小作为第二个参数传入即可，比如上面这个例子就创建了一个大小为1024的int类型channel，即使没有读取方，写入方也可以一直往channel里写入，在缓冲区被填完之前都不会阻塞。

从带缓冲的channel中读取数据可以使用与常规非缓冲channel完全一致的方法，但我们也可以使用range关键来实现更为简便的循环读取：

```go
for i := range c {
    fmt.Println("Received:", i)
}
package main

import (
	"fmt"
	"time"
)

func A(c chan int){
	for i:=0;i<10;i++{
		c<- i
	}
}
func B(c chan int){
	for val:=range c {
		fmt.Println("Value:",val)
	}
}
func main(){
	chs:=make(chan int,10)
	//只要有通道操作一定要放到goroutine中否则会堵塞当前的主线程 并且导致程序退出
	//对于同步通道 或者带缓冲的通道 一定要封装成函数 使用 goroutine 包装
	go A(chs)
	go B(chs)
	time.Sleep(1e9)
}
```

# 9、关于创建多个goroutine具体到go语言会创建多少个线程

```go
package main

import (
	//"fmt"
	//"time"
	"os"
)

func main() {
	for i:=0; i<20; i++ {
		go func() {
			for {
				b:=make([]byte, 10)
				os.Stdin.Read(b) // will block
			}
		}()
	}
	select{}
}
```

会产生21个线程：
runtime scheduler(src/pkg/runtime/proc.c)会***\*维护一个线程池，当某个goroutine被block后，scheduler会创建一个新线程给其他ready的goroutine。\****
GOMAXPROCS控制的是未被阻塞的所有goroutine被multiplex到多少个线程上运行

# 10、在channel中也是可以传递channel的,Go语言的channel和map  slice等一样都是原生类型

```go
[]chan int   chan int切片    chan []int 表示的是chan里面存的数据时[]int
```

需要注意的是，在Go语言中channel本身也是一个原生类型，与map之类的类型地位一样，因此channel本身在定义后也可以通过channel来传递。

我们可以使用这个特性来实现*nix上非常常见的管道（pipe）特性。管道也是使用非常广泛的一种设计模式，比如在处理数据时，我们可以采用管道设计，这样可以比较容易以插件的方式增加数据的处理流程。

下面我们***\*利用channel可被传递的特性来实现我们的管道\****。 为了简化表达， 我们假设在管道中传递的数据只是一个整型数，在实际的应用场景中这通常会是一个数据块。

首先限定基本的数据结构：



```go
type PipeData struct {
    value int
    handler func(int) int
    next chan int
}
```

然后我们写一个常规的处理函数。我们只要定义一系列PipeData的数据结构并一起传递给 这个函数，就可以达到流式处理数据的目的：

```go
func handle(queue chan *PipeData) {
for data := range queue {
        data.next <- data.handler(data.value)
    }
}
```

# 11、我们默认创建的是双向通道,单向通道没有意义,但是我们却可以通过强制转换 将双向通道 转换成为单向通道 



```go
var ch1 chan int  // ch1是一个正常的channel，不是单向的  
var ch2 chan<- float64// ch2是单向channel，只用于写float64数据
var ch3 <-chan int // ch3是单向channel，只用于读取int数据 
```

channel是一个原生类型，因此不仅支持被传递，还支持类型转换。只有在介绍了单向channel的概念后，读者才会明白类型转换对于channel的意义：就是在单向channel和双向channel之间进行转换。

示例如下：



```go
ch4 := make(chan int)
ch5 := <-chan int(ch4) // ch5就是一个单向的读取channel
ch6 := chan<- int(ch4) // ch6 是一个单向的写入channel
```

基于ch4，我们通过类型转换初始化了两个单向channel：单向读的ch5和单向写的ch6。 

从设计的角度考虑，所有的代码应该都遵循"最小权限原则" ，从而避免没必要地使用泛滥问题， 进而导致程序失控。 写过C++程序的读者肯定就会联想起const 指针的用法。非const指针具备const指针的所有功能，将一个指针设定为const就是明确告诉 

***\*函数实现者不要试图对该指针进行修改。单向channel也是起到这样的一种契约作用\****。下面我们来看一下单向channel的用法：



```go
func Parse(ch <-chan int) {
for value := range ch {
        fmt.Println("Parsing value", value)  
    }
}
```

除非这个函数的实现者无耻地使用了类型转换，否则这个函数就不会因为各种原因而对ch 进行写，避免在ch中出现非期望的数据，从而很好地实践最小权限原则。

# 12、只读只写 单向 channel 代码例子   遵循权限最小化的原则

```go
package main

import (
	"fmt"
	"time"
)
//接受一个参数 是只允许读取通道  除非直接强制转换 要么你只能从channel中读取数据
func sCh(ch <-chan int){
	for val:= range ch {
		fmt.Println(val)
	}
}
func main(){
	//创建一个带100缓冲的通道 可以直接写入 而不会导致 主线程堵塞
	dch:=make(chan int,100)
	for i:=0;i<100;i++{
		dch<- i
	}
	//传递进去 只读通道
	go sCh(dch)
	time.Sleep(1e9)
}
```

# 13、channel的关闭,以及判断channel的关闭



关闭channel非常简单，直接使用Go语言内置的close()函数即可：



```sql
close(ch)
```

在介绍了如何关闭channel之后，我们就多了一个问题：如何判断一个channel是否已经被关闭？我们可以在读取的时候使用多重返回值的方式：



```haskell
x, ok := <-ch
```

这个用法与map中的按键获取value的过程比较类似，只需要看第二个bool返回值即可，如果返回值是false则表示ch已经被关闭。

# 14、Go的多核并行化编程   高性能并发编程 必须设置GOMAXPROCS 为最大核数目 这个值由runtime.NumCPU()获取

在执行一些昂贵的计算任务时， 我们希望能够尽量***\*利用现代服务器普遍具备的多核特性来尽量将任务并行化\****，从而达到降低总计算时间的目的。此时我们需要了解***\*CPU核心的数量\****，并针对性地分解计算任务到多个goroutine中去并行运行。

下面我们来模拟一个完全可以并行的计算任务：计算N个整型数的总和。我们可以将***\*所有整型数分成M份，M即CPU的个数\****。让每个CPU开始计算分给它的那份计算任务，最后将每个CPU的计算结果再做一次累加，这样就可以得到所有N个整型数的总和：



```haskell
type Vector []float64
// 分配给每个CPU的计算任务
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
for ; i < n; i++ {
         v[i] += u.Op(v[i])
     }
     c <- 1       
// 发信号告诉任务管理者我已经计算完成了
}
const NCPU = 16     
// 假设总共有16核   
func (v Vector) DoAll(u Vector) {   
    c := make(chan int, NCPU)  // 用于接收每个CPU的任务完成信号   
    for i := 0; i < NCPU; i++ {   
            go v.DoSome(i*len(v)/NCPU, (i+1)*len(v)/NCPU, u, c)
    } 
    // 等待所有CPU的任务完成
    for i := 0; i < NCPU; i++ {   
         <-c    // 获取到一个数据，表示一个CPU计算完成了
    }
// 到这里表示所有计算已经结束
}
```

这两个函数看起来设计非常合理。***\*DoAll()会根据CPU核心的数目对任务进行分割\****，然后开辟多个goroutine来并行执行这些计算任务。是否可以将总的计算时间降到接近原来的1/N呢？答案是不一定。如果掐秒表（正常点的话，应该用7.8节中介绍的Benchmark方法） ，会发现总的执行时间没有明显缩短。再去观察CPU运行状态， 你会发现***\*尽管我们有16个CPU核心， 但在计算过程中其实只有一个CPU核心处于繁忙状态\****，这是会让很多Go语言初学者迷惑的问题。

官方的答案是，这是当前版本的Go编译器还不能很智能地去发现和利用多核的优势。虽然我们确实创建了多个goroutine，并且从运行状态看这些goroutine也都在并行运行，但实际上***\*所有这些goroutine都运行在同一个CPU核心上， 在一个goroutine得到时间片执行的时候， 其他goroutine都会处于等待状态\****。从这一点可以看出，虽然goroutine简化了我们写并行代码的过程，但实际上整体运行效率并不真正高于单线程程序。

在Go语言升级到默认支持多CPU的某个版本之前，我们可以先通过设置环境变量

GOMAXPROCS的值来控制使用多少个CPU核心。具体操作方法是通过直接设置环境变量GOMAXPROCS的值，或者在代码中启动goroutine之前先调用以下这个语句以设置使用16个CPU核心：



```undefined
runtime.GOMAXPROCS(16)
```

到底应该设置多少个CPU核心呢，其实runtime包中还提供了另外一个***\*函数NumCPU()来获取核心数\****。可以看到，Go语言其实已经感知到所有的环境信息，下一版本中完全可以利用这些信息将goroutine调度到所有CPU核心上，从而最大化地利用服务器的多核计算能力。抛弃GOMAXPROCS只是个时间问题。



# 15、主动出让时间片给其他 goroutine 在未来的某一时刻再来执行当前goroutine

我们可以在每个goroutine中控制何时主动出让时间片给其他goroutine，这可以使用***\*runtime包中的Gosched()函数\****实现。
实际上，如果要比较精细地控制goroutine的行为，就必须比较深入地了解Go语言开发包中runtime包所提供的具体功能。



# 16、Go中的同步及同步锁

倡导用通信来共享数据，而不是通过共享数据来进行通信，但考虑到即使成功地用channel来作为通信手段，还是***\*避免不了多个goroutine之间共享数据的问题\****，Go语言的设计者虽然对channel有极高的期望，但也提供了妥善的***\*资源锁方案\****。

对于这两种锁类型， 任何一个Lock()或RLock()均需要保证对应有Unlock()或RUnlock()调用与之对应，***\*否则可能导致等待该锁的\*******\*所有goroutine处于饥饿状态，甚至可能导致死锁\****。锁的典型使用模式如下：



```swift
var l sync.Mutex  
func foo() {
l.Lock()  
//延<strong>迟调用 在函数退出 并且局部资源被释放的时候 调用</strong>
defer l.Unlock()  
//...
}  
```

这里我们再一次见证了Go语言defer关键字带来的优雅。

# 17、全局唯一操作 sync.Once.Do() sync.atomic原子操作子包



对于从全局的角度只需要运行一次的代码，比如全局初始化操作，Go语言提供了一个***\*Once类型来保证全局的唯一性操作\****，具体代码如下：



```go
var a string
var once sync.Once  
func setup() {
    a = "hello, world"
}  
func doprint() {
    once.Do(setup)
    print(a)  
}  
func twoprint() {
    go doprint()
    go doprint()  
}
```

如果这段代码没有引入Once， setup()将会被每一个goroutine先调用一次， 这至少对于这个例子是多余的。在现实中，我们也经常会遇到这样的情况。Go语言标准库为我们引入了Once类型以解决这个问题。***\*once的Do()方法可以保证在全局范围内只调用指定的函数一次\****（这里指setup()函数），而且所有其他goroutine在调用到此语句时，将会先被阻塞，直至全局唯一的once.Do()调用结束后才继续。

这个机制比较轻巧地解决了使用其他语言时开发者不得不自行设计和实现这种Once效果的难题，也是Go语言为并发性编程做了尽量多考虑的一种体现。

如果没有once.Do()，我们很可能只能添加一个全局的bool变量，在函数setup()的最后一行将该bool变量设置为true。在对setup()的所有调用之前，需要先判断该bool变量是否已经被设置为true，如果该值仍然是false，则调用一次setup()，否则应跳过该语句。实现代码



```go
var done bool = false
func setup() {
    a = "hello, world" 
    done = true
}     
func doprint() { 
    if !done {
        setup()
    }   
    print(a)  
} 
```

这段代码初看起来比较合理， 但是细看还是会有问题， 因为***\*setup()并不是一个原子性操作\****，这种写法可能导致setup()函数被多次调用，从而无法达到全局只执行一次的目标。这个问题的复杂性也更加体现了Once类型的价值。 //还没有执行到done=true的时候  另外调用doprint已经执行起来。

为了更好地控制并行中的原子性操作，sync包中还包含一个atomic子包，它提供了对于一些基础数据类型的原子操作函数，比如下面这个函数：



```go
func CompareAndSwapUint64(val *uint64, old, new uint64) (swapped bool)
```

就提供了比较和交换两个uint64类型数据的操作。这让开发者无需再为这样的操作专门添加Lock操作。

转载链接：http://lib.csdn.net/article/53/36140?knId=1441