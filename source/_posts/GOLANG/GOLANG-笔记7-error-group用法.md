---
title: GOLANG-笔记7-error.group用法
categories:
  - GOLANG
date: 2024-03-26 14:18:21
tags: error.group
cover:  /img/golang.jpeg

---

#### 1 .先看下数据结构

```go
type Group struct {
	cancel func()

	wg sync.WaitGroup

	sem chan token

	errOnce sync.Once
	err     error
}
```

#### 2.在并发编程里 sync.WaitGroup

并发原语的使用频率非常高，它经常用于协同等待的场景 gorouting 都完成后才能继续执行。

如果在woker goroutine的执行过程中遇到错误并想要处理该怎么办？

WaitGroup并没有提供传播错误的功能，遇到这种场景我们该怎么办？Go语言在扩展库提供了ErrorGroup并发原语正好适合在这种场景下使用，它在WaitGroup的基础上还提供了，错误传播以及上下文取消的功能。

扩展库通过errorgroup.Group提供ErrorGroup原语的功能，它有三个方法可调用

```
func WithContext(ctx context.Context) (*Group, context.Context)
func (g *Group) Go(f func() error)
func (g *Group) Wait() error
```

```go
//ErrorGroup有一个特点是会返回所以执行任务的goroutine遇到的第一个错误
```

#### 3.想让程序遇到错误就终止其他子任务

最早执行遇到错误的goroutine输出了Error: 98但是所有未执行完的其他任务并没有停止执行，那么想让程序遇到错误就终止其他子任务该怎么办呢？我们可以用errgroup.Group提供的WithContext方法创建一个带可取消上下文功能的ErrorGroup。

```
/**
使用errorgroup.Group时注意它的两个特点：
- errgroup.Group在出现错误或者等待结束后都会调用 Context对象 的 cancel 方法同步取消信号。
- 只有第一个出现的错误才会被返回，剩余的错误都会被直接抛弃。
*/
func main() {
   eg, ctx := errgroup.WithContext(context.Background())

   for i := 0; i < 100; i++ {
      i := i
      eg.Go(func() error {
         time.Sleep(2 * time.Second)

         select {
         case <-ctx.Done():
            fmt.Println("Canceled:", i)
            return nil
         default:
            if i > 90 {
               fmt.Println("Error:", i)
               return fmt.Errorf("Error: %d", i)
            }
            fmt.Println("End:", i)
            return nil
         }
      })
   }
   if err := eg.Wait(); err != nil {
      log.Fatal(err)
   }
}

```



####  4.cancle到其他的子任务

在上面的例子中，子goroutine出现错误后，会cancle到其他的子任务，但是我们并没有看到调用ctx的cancel方法，下面我们看下源码，看看内部是怎么处理的。
 errgroup 的设计非常精练，全部代码如下

```
package errgroup

import (
    "context"
    "sync"
)

// A Group is a collection of goroutines working on subtasks that are part of
// the same overall task.
//
// A zero Group is valid and does not cancel on error.
type Group struct {
    cancel func()
    wg sync.WaitGroup
    errOnce sync.Once
    err     error
}

// WithContext returns a new Group and an associated Context derived from ctx.
//
// The derived Context is canceled the first time a function passed to Go
// returns a non-nil error or the first time Wait returns, whichever occurs
// first.
func WithContext(ctx context.Context) (*Group, context.Context) {
    ctx, cancel := context.WithCancel(ctx)
    return &Group{cancel: cancel}, ctx
}

// Wait blocks until all function calls from the Go method have returned, then
// returns the first non-nil error (if any) from them.
func (g *Group) Wait() error {
    g.wg.Wait()
    if g.cancel != nil {
        g.cancel()
    }
    return g.err
}

// Go calls the given function in a new goroutine.
//
// The first call to return a non-nil error cancels the group; its error will be
// returned by Wait.
func (g *Group) Go(f func() error) {
    g.wg.Add(1)

    go func() {
        defer g.wg.Done()

        if err := f(); err != nil {
            g.errOnce.Do(func() {
                g.err = err
                if g.cancel != nil {
                    g.cancel()
                }
            })
        }
    }()
}

```



可以看到，errgroup 的实现依靠于结构体 Group，它通过封装 sync.WaitGroup，继承了 WaitGroup 的特性，在 Go() 方法中新起一个子任务 goroutine，并在 Wait() 方法中通过 sync.WaitGroup 的 Wait 进行阻塞等待。

同时 Group 利用 sync.Once 保证了它有且仅会保留第一个子 goroutine 错误。
 Group 通过嵌入 context.WithCancel 方法产生的 `cancel` 函数（对于 Context 不熟悉的读者，推荐阅读 [理解Context机制](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzkyMzIyNjIxMQ%3D%3D%26mid%3D2247484553%26idx%3D1%26sn%3Dce4c7e052bacf69c38e27d71187726c8%26scene%3D21%23wechat_redirect) 一文），能够**在子 goroutine 发生错误时，及时通过调用 cancle 函数，将 Context 的取消信号及时传播出去。**

#### 5.总结:

使用errorgroup.Group时注意它的特点：

- 继承了 WaitGroup 的功能
- errgroup.Group在出现错误或者等待结束后都会调用Context对象 的 cancel 方法同步取消信号。
- 只有第一个出现的错误才会被返回，剩余的错误都会被直接抛弃。
- context 信号传播：如果子任务 goroutine 中有循环逻辑，则可以添加 ctx.Done 逻辑，此时通过 context 的取消信号，提前结束子任务执行。

































.

















