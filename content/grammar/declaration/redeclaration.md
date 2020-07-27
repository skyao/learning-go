---
date: 2020-07-04T23:50:00+08:00
title: 重新声明
weight: 458
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的重新声明"
---

### Redeclaration and reassignment

https://golang.org/doc/effective_go.html#redeclaration

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

旁白一下。上一节的最后一个例子演示了 := 短声明形式的工作细节。调用os.Open的声明如下。

```go
f, err := os.Open(name)
```

这条语句声明了两个变量，f和err。几行之后，对f.Stat的调用为：

```go
d, err := f.Stat()
```

这看起来就像它声明了d和err。但请注意，两个语句中都出现了err。这种重复是合法的：err 是由第一条语句声明的，但只是在第二条语句中重新赋值。这意味着对 f.Stat 的调用使用了上面声明的现有err变量，只是给它一个新的值。

在 := 声明中，即使已经声明了一个变量 v，也可以出现，但前提是：

- 这个声明与v的现有声明在同一个作用域中（如果v已经在外部作用域中声明了，则声明将创建一个新的变量§）。
- 初始化中的相应值可分配给v，并且
- 至少还有一个变量是由声明创建的。

这个不寻常的属性是纯粹的实用主义，使得它很容易使用一个单一的err值，例如，在一个长的if-else链中。你会看到它经常被使用。

§ 这里值得注意的是，在Go中，函数参数和返回值的作用域与函数主体相同，尽管它们在词法上出现在包围主体的括号之外。