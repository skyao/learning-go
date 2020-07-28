---
date: 2020-07-04T23:50:00+08:00
title: for语句
weight: 603
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 for 语句"
---

## go语言实战

### for loop

Go 只有一种循环结构 `for` 循环。

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

和Java的语法相反：

1. for 后面没有括号`()`，注意是强制一定不能有
2. 循环体必须有`{}`

跟 C 或者 Java 中一样，可以让前置、后置语句为空：

```go
sum := 1
for ; sum < 1000; {
    sum += sum
}
```

这就非常类似C、Java中的while循环了，因此干脆继续简写，省略掉分括号：

```go
sum := 1
for sum < 1000 {
    sum += sum
}
```

更绝一点，死循环：

```go
for {
}
```

## golang语言规范

https://golang.org/ref/spec#For_statements

"for "语句指定重复执行块。有三种形式。迭代可以由一个条件(condition)、一个 "for "子句或一个 "range "子句控制。

```
ForStmt = "for" [ Condition | ForClause | RangeClause ] Block .
Condition = Expression .
```

### 单一条件的 for 语句

在最简单的形式中，"for "语句指定重复执行一个块，只要一个布尔条件评估为真。该条件在每次迭代前都会被评估。如果条件不存在，则相当于布尔值为true。

```go
for a < b {
	a *= 2
}
```

### 带有for子句的for语句

带有 ForClause 的 "for "语句也是由它的条件(condition)控制的，但除此之外，它还可以指定一个 init 和一个post语句，如赋值、增量或减量语句。init 语句可以是一个简短的变量声明，但post语句不能。init语句声明的变量在每次迭代中都会被重复使用。

```
ForClause = [ InitStmt ] ";" [ Condition ] ";" [ PostStmt ] .
InitStmt = SimpleStmt .
PostStmt = SimpleStmt .
```

```go
for i := 0; i < 10; i++ {
	f(i)
}
```

如果非空，则在评估第一次迭代的条件之前，init语句会被执行一次；post语句会在每次执行块之后执行（而且只有在块被执行的情况下）。ForClause 的任何元素都可以是空的，但分号是必须的，除非只有一个条件。如果条件不存在，则相当于布尔值为true。

```
for cond { S() }    is the same as    for ; cond ; { S() }
for      { S() }    is the same as    for true     { S() }
```

## Effective Go

https://golang.org/doc/effective_go.html#for

Go for循环与C的循环类似-但不一样。它统一了for和while，而且没有do-while。有三种形式，其中只有一种有分号。

```go
// Like a C for （除了不容许用括号）
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

短声明使其很容易在循环中直接声明索引变量。

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

最后，Go没有逗号运算符，++和--是语句而不是表达式。因此，如果你想在for中运行多个变量，你应该使用并行赋值（尽管这排除了++和--）。

```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

