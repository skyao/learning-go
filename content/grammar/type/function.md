---
date: 2020-07-04T23:50:00+08:00
title: 函数类型
weight: 478
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的函数类型"
---

## Function types

https://golang.org/ref/spec#Function_types

函数类型表示具有相同参数和结果类型的所有函数的集合。未初始化的函数类型变量的值为零。

```
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

在参数或结果的列表中，名称（IdentifierList）必须全部存在或全部不存在。如果存在，每个名称代表指定类型的一个项目（参数或结果），并且签名中所有非空白名称必须是唯一的。如果不存在，每个类型代表该类型的一个项目。参数和结果列表总是用括号表示，但如果正好有一个未命名的结果，则可以写成一个未括号的类型。

在函数签名中，最后一个输入的参数可以有一个以...为前缀的类型。带有这样参数的函数被称为 *variadic* 可变参数，可以用零或多个参数来调用该参数。

```go
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```

