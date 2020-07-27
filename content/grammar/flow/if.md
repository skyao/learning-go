---
date: 2020-07-04T23:50:00+08:00
title: If语句
weight: 601
menu:
  main:
    parent: "grammar-flow"
description : "go语言类型中的if语句"
---

## If语句

类似for循环，go中的if语句也是同样，if后面不能有括号，而if里面必须要有花括号：

```go
if x < 0 {
    return sqrt(-x) + "i"
}
```

跟 `for` 一样，`if` 语句可以在条件之前执行一个简单的语句：

```go
if v := math.Pow(x, n); v < lim {
    // v在这里可以访问
    return v
}
// v在这里不可以访问
```

注意：这个语句定义的变量的作用域仅在 `if` 范围之内，包括else：

```go
if v := math.Pow(x, n); v < lim {
    return v
} else {
    // else这里可以访问
    fmt.Printf("%g >= %g\n", v, lim)
}
```

## If statements

https://golang.org/ref/spec#If_statements

"if "语句根据布尔表达式的值指定两个分支的条件执行。如果表达式的值为真，则执行 "if "分支，否则，如果存在，则执行 "else "分支。

```
IfStmt = "if" [ SimpleStmt ";" ] Expression Block [ "else" ( IfStmt | Block ) ] .
```

```
if x > max {
	x = max
}
```

表达式前面可以有一个简单的语句，在表达式被评估之前执行。

```go
if x := f(); x < y {
	return x
} else if x > z {
	return z
} else {
	return y
}
```

## if

https://golang.org/doc/effective_go.html#if

在围棋中，一个简单的if是这样的：

```go
if x > 0 {
    return y
}
```

强制括号鼓励在多行上写简单的if语句。无论如何这样做都是好的风格，特别是当正文中包含一个控制语句，如返回或break时。

由于if和switch接受一个初始化语句，所以通常会看到一个用来设置局部变量的语句。

```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在 Go 库中，你会发现，当一个 if 语句没有流入下一个语句时--也就是说，正文以 break、continue、goto 或 return 结尾时，不必要的 else 会被省略。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

这是一个常见情况的例子，代码必须防范一连串的错误情况。如果成功的控制流向下运行，在出现错误情况时消除错误情况，那么代码的阅读效果就会很好。由于错误情况往往以return语句结束，因此产生的代码不需要 else语句。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

