---
date: 2020-07-04T23:50:00+08:00
title: select语句
weight: 605
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 select 语句"
---

## go语言实战

select是Golang中的控制语句，语法类似于switch语句。但是select只用于通信，要求每个case必须是IO操作。

不带default语句的select会阻塞直到某个case满足：

```go
ch1 := make (chan int, 1)
ch2 := make (chan int, 1)

select {
case <-ch1:
    fmt.Println("ch1 pop one element")
case <-ch2:
    fmt.Println("ch2 pop one element")
}
```

如果两个case同时满足，则随机执行某个case语句，其他case语句不会执行。 

如果不想阻塞，则可以带上default子语句：

```go
select {
case <-ch1:
    fmt.Println("ch1 pop one element")
case <-ch2:
    fmt.Println("ch2 pop one element")
default:
    fmt.Println("not ready yet")
}
```

如果两个case条件都不满足，则直接跳到 default 流程而不阻塞。

## golang语言规范

https://golang.org/ref/spec#Select_statements

“select"语句选择一组可能的发送或接收操作中的某一个进行。它看起来类似于 "switch"语句，但情况都是指通信操作（指IO操作）。

```
SelectStmt = "select" "{" { CommClause } "}" .
CommClause = CommCase ":" StatementList .
CommCase   = "case" ( SendStmt | RecvStmt ) | "default" .
RecvStmt   = [ ExpressionList "=" | IdentifierList ":=" ] RecvExpr .
RecvExpr   = Expression .
```

带有RecvStmt的情况下，可以将RecvExpr的结果分配给一个或两个变量，这些变量可以使用短变量声明。RecvExpr必须是一个（可能是括号）接收操作。最多只能有一个default缺省情况，它可以出现在case列表中的任何地方。

"select "语句的执行分几个步骤进行：

1. 对于语句中的所有case，在进入 "select "语句后，接收操作的通道操作数和发送语句的通道和右侧表达式都会按照源代码顺序精确地评估一次。其结果是一组可以接收或发送的通道，以及相应的发送值。无论选择哪种（如果有的话）通信操作进行，该评估中的任何副作用都会发生。RecvStmt左侧带有短变量声明或赋值的表达式尚未被评估。

2. 如果有一个或多个通信可以进行，则通过统一的伪随机选择选择一个可以进行的单一通信。否则，如果有默认case，则选择该case。如果没有default case，"select "语句就会阻塞，直到至少有一个通信可以继续进行。

3. 除非选择的case是默认情况，否则会执行相应的通信操作。

4. 如果选择的case是一个带有短变量声明或赋值的RecvStmt，则左手边的表达式被评估，接收到的值（或数值）被赋值。

5. 所选case的语句列表被执行。

由于在nul通道上的通信永远无法进行，所以只有nil通道而没有 default case 的 select 永远阻塞。

```go
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
case i1 = <-c1:
	print("received ", i1, " from c1\n")
case c2 <- i2:
	print("sent ", i2, " to c2\n")
case i3, ok := (<-c3):  // same as: i3, ok := <-c3
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// same as:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}

for {  // send random sequence of bits to c
	select {
	case c <- 0:  // note: no statement, no fallthrough, no folding of cases
	case c <- 1:
	}
}

select {}  // block forever
```











