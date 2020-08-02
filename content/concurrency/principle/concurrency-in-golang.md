---
date: 2020-07-04T23:50:00+08:00
title: Concurrency in Golang
weight: 1214
menu:
  main:
    parent: "concurrency-principle"
description : "go语言中的并发"
---

http://www.minaandrawos.com/2015/12/06/concurrency-in-golang/

昨天，我在Quora里回答了一个关于Go中并发模型的问题。现在，我觉得我想多说几句! Golang中的并发是该语言中最强大的特性之一。众多的人都涉及到了这个话题，他们的看法从非常简单到过于复杂不等。今天，轮到我说说我的看法了。

Golang中的并发是一种思维方式，不仅仅是语法。为了利用Go的力量 ，你需要先了解Go是如何处理代码的并发执行的。Go依赖于一个名为CSP ( Communicating Sequential Processes)的并发模型，在计算机科学中，CSP基本上是一个描述并发系统之间交互的模型。但由于这不是一篇科学论文，我将跳过这些形式化的东西，跳到它的实际描述中去。

在Golang中解释并发性时，很多Go讲座、演示和文档都会使用以下描述:

> **Do not communicate by sharing memory; instead, share memory by communicating.** 
>
> **不要通过共享内存来通信，而是通过通信来共享内存。**

听起来不错，不错。但这到底是什么意思呢。我花了一段时间才用脑子完全理解这个概念。但一旦我做到了，用Go编程对我来说就变得更加流畅了。阿尔伯特-爱因斯坦曾经说过："如果你不能简单地解释它，你就不能很好地理解它。" ，所以这是我能想到的最简单的解释。

### 不要通过共享内存进行通信

在主流的编程语言中，当你想到代码的并发执行时，你大多会想到一堆线程并行运行，执行某种复杂的操作。那么，大多数情况下，你需要在不同的线程之间共享数据结构/变量/内存/什么的。你可以通过锁定这块内存来实现，这样就不会有两个线程同时访问/写入它，或者你只是让它自由漫游，并希望得到最好的结果。在很多流行的编程语言中，这通常是不同线程 "通信"的方式，这通常会导致各种问题，如竞争条件，内存管理 ，随机-奇怪-无法解释的异常-唤醒-你-整夜......等等。

### 而是通过通信来共享内存

那么Go是怎么做的呢？Go不锁定变量来共享内存，而是允许将存储在变量中的值从一个线程通信（或发送）到另一个线程（实际上，这并不完全是一个线程，但我们现在就把它当作一个线程）。默认的行为是发送数据的线程和接收数据的线程都会等待，直到值到达目的地。线程的 "等待"迫使线程之间在交换数据时进行适当的同步。在挽起袖子开始设计代码之前，先这样思考并发性问题；可以让软件更加稳定。

更明确的说：稳定性来自于这样一个事实 —— 默认情况下，发送线程和接收线程都不会做任何事情，直到值传输完成。意思是说，在另一个线程完成数据传输之前，任何一个线程对数据进行操作，都不会出现竞争条件或类似的问题。

Go提供了原生特性，你可以使用这些特性来实现这种行为，而不需要调用额外的类库或框架，这种行为只是简单地内置在语言中。如果你需要的话，Go还允许你拥有一个 "缓冲通道"。这意味着在某些情况下，你不希望两个线程都锁定或同步，直到一个值被传输，相反，你希望只有当你在两个线程之间的通道中填满了预定义数量的待处理值时，同步/锁定才会发生。

不过需要提醒的是，这种模式可能会被过度使用。你必须感觉到什么时候使用它，或者什么时候恢复到老式的共享内存模型。例如，引用计数最好在锁里面保护，文件访问也是如此。Go也会通过同步包在那里支持你。

### golang中的并发编码

那么我们就来聊聊代码吧。我们如何实现逐个通信模式的共享呢？ 请继续阅读

在Go中，"goroutine"服务于我们上面描述的线程的概念。实际上，它并不是真正的线程，它基本上是一个函数，它可以和其他goroutine在同一个地址空间中并发运行。它们在O.S.线程之间是多路复用的，所以如果一个线程阻塞，其他线程可以继续运行。所有的同步和内存管理都是由Go原生完成的。它们之所以不是真正的线程，是因为它们不一定是一直并行的。然而，由于多路复用和同步，你会得到并发行为。要启动一个新的goroutine，你只需要使用关键字 "go"。

```go
go processdataFunction()
```

"go channel"是Go中实现并发的另一个关键概念。这是用于在goroutine之间进行内存通信的通道。要创建一个通道，可以使用 "make"。

```go
myChannel := make(chan int64)
```

创建一个缓冲通道，以便在goroutines等待之前允许更多的值被排队，看起来像这样：

```go
myBufferedChannel := make(chan int64,4)
```

在上面的两个例子中，我假设在这之前没有创建通道变量。这就是为什么我使用":="来创建具有推断类型的变量，而不是"="，因为"="只会赋值，而且如果之前没有声明该变量，将导致编译错误。

现在要使用一个通道，你可以使用"<-"符号。goroutine发送的值会像这样分配给通道。

```go
mychannel <- 54
```

接收数值的goroutine将从通道中提取数值，并将其分配到一个新的变量中，就像这样：

```go
myVar := <- mychannel
```

现在让我们看一个例子来展示Golang中的并发性：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
	ch := make(chan int)
	//create a channel to prevent the main program from exiting before the done signal is received
	done := make(chan bool)
	go sendingGoRoutine(ch)
	go receivingGoRoutine(ch,done)
	//This will prevent the program from exiting till a value is sent over the "done" channel, value doesn't matter
	<- done
}

func sendingGoRoutine(ch chan int){
	//start a timer to wait 5 seconds
	t := time.NewTimer(time.Second*5)
	<- t.C
	fmt.Println("Sending a value on a channel")
    //this goroutine will wait till another goroutine received the value
    ch <- 45
}

func receivingGoRoutine(ch chan int, done chan bool){
	//this gourtine will wait till the channel received a value
    v := <- ch
	fmt.Println("Received value ", v)
	done <- true
}
```

输出将是这样的：

```go
Sending a value on a channel
Received value  45
```

















