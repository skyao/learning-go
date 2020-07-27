---
date: 2020-07-04T23:50:00+08:00
title: Channel类型
weight: 481
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的Channel类型"
---

## Channel types

https://golang.org/ref/spec#Channel_types

通道为并发执行的函数提供了一种机制，通过发送和接收指定元素类型的值进行通信。未初始化的通道的值为nil。

```
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

可选的 `<-` 操作符指定通道方向，发送或接收。如果没有给出方向，则通道是双向的。通道可以通过赋值或显式转换来限制只能发送或只能接收。

```go
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints
```

<-操作符与最左边的chan可能关联。

```go
chan<- chan int    // same as chan<- (chan int)
chan<- <-chan int  // same as chan<- (<-chan int)
<-chan <-chan int  // same as <-chan (<-chan int)
chan (<-chan int)
```

可以使用内置的函数make制作一个新的、初始化的通道值，该函数将通道类型和一个可选的容量作为参数。

```go
make(chan int, 100)
```

capacity，以元素数为单位，设置通道中缓冲区的大小。如果容量为零或不存在，则通道是无缓冲的，只有当发送方和接收方都准备好时，通信才会成功。否则，通道被缓冲，如果缓冲区没有满（发送）或不空（接收），则通信成功而不阻塞。nil通道永远不会为通信做好准备。

可以用内置函数close关闭通道。接收运算符的多值赋值形式报告在通道关闭前是否有接收值被发送。

单个通道可以用于发送语句、接收操作，以及任何数量的goroutine对内置函数cap和len的调用，而无需进一步同步。通道是先入先出(first-in-first-out)的队列。例如，如果goroutine在通道上发送数值，而第二个goroutine接收数值，那么数值将按照发送的顺序接收。

