---
date: 2020-07-04T23:50:00+08:00
title: go语句
weight: 604
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 go 语句"
---

## golang语言规范

https://golang.org/ref/spec#Go_statements

"go"语句在同一地址空间内，以独立的并发控制线程或goroutine的形式开始执行一个函数调用。

```
GoStmt = "go" Expression .
```

表达式必须是函数或方法调用，不能用括号。内建函数的调用与表达式语句一样受到限制。

**函数值和参数在调用goroutine中像往常一样被评估**，但与常规调用不同的是，程序的执行不会等待被调用的函数完成，而是在新的goroutine中开始独立执行。相反，函数在新的goroutine中开始独立执行。当函数终止时，它的goroutine也会终止。如果函数有任何返回值，则在函数完成时将其丢弃。

```go
go Server()
go func(ch chan<- bool) { for { sleep(10); ch <- true }} (c)
```

