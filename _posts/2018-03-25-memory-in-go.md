---
layout: post
title:  "GoLang内存模型"
date:   2018-03-25 20:17:59
categories: 
- code 
tags:
- go
- memory 
---
Go 内存模型阐明了一个 Go 程序对某变量的写入, 如何才能确保另一个读取该变量的 Go 程序监测到 

#### 忠告

* 程序在修改被多个 GO 程序同时访问的数据时必须序列化操作
* 要序列化操作, 需要通过信道, 或者使用 **sync** 和 **sync/atomic** 包中的同步原语来进行数据保护 

#### 事件发生次序

> 在单个Go程中，读取和写入的表现必须与程序指定的执行顺序相一致。换言之， 仅在不会改变语言规范对Go程行为的定义时，编译器和处理器才会对读取和写入的执行重新排序。 
>
> 为了详细论述读取和写入的必要条件，我们定义了**事件发生顺序**，若事件 e1 发生在 e2 之前， 那么我们就说 e2 发生在 e1 之后。 换言之，若 e1 既未发生在 e2 之前， 又未发生在 e2 之后，那么我们就说 e1 与 e2 是并发的。

#### 同步 

##### 初始化

> 程序的初始化运行在单个 Go程中，但该Go程可能会创建其它并发运行的Go程。
>
> 若包 `p` 导入了包 `q`，则 `q` 的 `init` 函数会在 `p` 的任何函数启动前完成。
>
> 函数 `main.main` 会在所有的 `init` 函数结束后启动。

##### Go 程的创建 

> go 语句会在当前 Go 程开始执行前启动新的 Go 程

```go
var a string
func f() { print(a) }
func hello() { 
	a = "hello"
  	go f()
} // 调用 hello 或许会在将来的某一时刻打印出 "hello" (在 hello 返回之后则会打印空值 )
```

##### Go 程的销毁

> 无法确保在程序中的任何事件发生之前退出

```go
var a string 
func hello() {
	go func() { a = "hello" }
  	print(a)
} // 对 a 赋值后并没有任何同步事件因此无法保证被其它任何go程检测到,实际上在这一个积极的编译器可能会删除整条 go 语句,若一个Go程的作用必须被另一个Go程监测到,需使用锁或信道通信之类的同步机制来建立顺序关系
```

##### 信道通信 

> 是 Go 程之间进行同步的主要方法, 信道上的发送操作总在对应的接收操作完成前发生 

```go
var c = make(chan int, 10)
var a string
func f() {
  	a = "hello"
  	c <- 0 // 此处可以用 close(c) 来代替, 也能保证程序产生相同的行为 
}
func main() {
  	go fo()
  	<- c
  	print(a)
} // 可保证打印出"hello", 先对a进行写入,然后在c上发送信息,随后从c接收信号进行阻塞,最后打印出a 
```

> 从无缓冲信道进行接收,要发生在对该信道进行的发送完成前 

```go
var c = make(chan int)
var a string
func f() {
  	a = "hello"
  	<- c
}
func main() {
  	go f()
  	c <- 0
  	print(a)
} // 此写法一样可以保证打印出"hello", 但如果信道是带缓冲的,则不能保证(它可能会打印出空字符串,崩溃或做些别的事情) 
```

##### 锁

> sync 包实现了两种锁的数据类型: sync.Mutex 跟 sync.RWMutex 
>
> 对于任何 `sync.Mutex` 或 `sync.RWMutex` 类型的变量 `l` 以及 *n* < *m* ，对 `l.Unlock()` 的第 *n* 次调用在对 `l.Lock()` 的第 *m* 次调用返回前发生

```go
var l sync.Mutex
var s string
func f() {
  	a = "hello"
  	l.Unlock()
}
func main() {
  	l.Lock()
  	go f() 
  	l.Lock()
  	print(a)
} // 可保证打印出"hello"
```

##### Once

> `sync` 包通过 `Once` 类型为存在多个Go程的初始化提供了安全的机制, 多个线程可为特定的 `f` 执行 `once.Do(f)`, 但只有一个会运行 `f()`, 而其它调用会一直阻塞, 直到 `f()` 返回

```go
var a string
var once sync.Once
var wg sync.WatiGroup
func init() {
  	wg.Add(2)
}
func setup() {
  	a = "hello"
  	print("run in setup func ...")
}
func doprint() {
  	defer wg.Done()
  	once.Do(setup)
  	print(a)
}
func main() {
	go doprint()
	go doprint()
	wg.Wait()
} // 可以看到会打印出两次 "hello", 但只会打印出一次 "run in setup func ..."
```

##### 错误的同步

> 读取操作 r 可能监测到与其并发的写入操作 w 写入的值。即便如此也并不意味着发生在 r 之后的读取操作会监测到发生在 w 之前的写入操作

