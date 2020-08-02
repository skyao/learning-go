---
date: 2020-07-04T23:50:00+08:00
title: 并发原则
weight: 1210
description : "介绍Go语言的并发原则"
---

> Do not communicate by sharing memory; instead, share memory by communicating.
>
> 不要通过共享内存来通讯，而是通过通讯来共享内存。

## Effective Go

https://golang.org/doc/effective_go.html#sharing

并发编程是一个很大的话题，这里仅有篇幅介绍一些Go特有的亮点。

在许多环境中，由于实现对共享变量的正确访问所需的微妙之处，使并发编程变得困难。Go鼓励一种不同的方法，其中共享值在通道上传递，事实上，从来没有被单独的执行线程主动共享。在任何时候，只有一个goroutine可以访问该值。通过设计，数据竞赛是不可能发生的。为了鼓励这种思维方式，我们将其简化为一句口号。

**不要通过共享内存进行通信，而是通过通信来共享内存。**

这种方法可能走得太远。比如说，引用计数可能最好的办法是在一个整数变量周围放一个mutex。但作为一种高级方法，使用通道来控制访问，可以更容易写出清晰、正确的程序。

思考这个模型的方法是考虑一个典型的单线程程序，它运行在一个CPU上。它不需要同步原语。现在运行另一个这样的实例；它也不需要同步。现在让这两个程序进行通信；如果通信的是同步器，那么仍然不需要其他同步。例如，Unix管道就完全符合这个模型。虽然Go的并发方法起源于Hoare的Communicating Sequential Processes(CSP)，但它也可以被看作是Unix管道的类型安全泛化。



### 学习资料

- [为什么使用通信来共享内存](https://draveness.me/whys-the-design-communication-shared-memory/)

