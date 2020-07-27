---
date: 2020-07-04T23:50:00+08:00
title: 变量声明
weight: 455
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的变量声明"
---

### Variable declaration

https://golang.org/ref/spec#Variable_declarations

变量声明创建一个或多个变量，将相应的标识符绑定到它们上面，并给每个变量一个类型和初始值。

```
VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec     = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .
```

```go
var i int
var U, V, W float64
var k = 0
var x, y float32 = -1, -2
var (
	i       int
	u, v, s = 2.0, 3.0, "bar"
)
var re, im = complexSqrt(-1)
var _, found = entries[name]  // map lookup; only interested in "found"
```

如果给定了一个表达式列表，则按照赋值规则用表达式初始化变量。否则，每个变量初始化为零值。

如果存在类型，每个变量被赋予该类型。否则，每个变量被赋予赋值中相应初始化值的类型。如果该值是一个无类型的常量，则首先隐式转换为其默认类型；如果是一个无类型的布尔值，则首先隐式转换为类型bool。预先声明的值nil不能用于初始化一个没有显式类型的变量。

```go
var d = math.Sin(0.5)  // d is float64
var i = 42             // i is int
var t, ok = x.(T)      // t is T, ok is bool
var n = nil            // illegal
```

实现限制：如果一个变量从未被使用，编译器可能会规定在函数体中声明一个变量是非法的。

### 短变量声明

短变量声明使用这样的语法：

```
ShortVarDecl = IdentifierList ":=" ExpressionList .
```

它是有初始化表达式但没有类型的正则变量声明的简写：

```
"var" IdentifierList = ExpressionList .
```

```go
i, j := 0, 10
f := func() int { return 7 }
ch := make(chan int)
r, w, _ := os.Pipe()  // os.Pipe() returns a connected pair of Files and an error, if any
_, y, _ := coord(p)   // coord() returns three values; only interested in y coordinate
```

与普通变量声明不同，短变量声明可以重新声明变量，但前提是这些变量原来在同一个块（如果块是函数体，则在参数列表中）中早先声明过，类型相同，而且至少有一个非空变量是新的。因此，重声明只能出现在多变量的短声明中。重新声明并不引入一个新的变量，它只是给原来的变量分配一个新的值。

```go
field1, offset := nextField(str, 0)
field2, offset := nextField(str, offset)  // redeclares offset
a, a := 1, 2                              // illegal: double declaration of a or no new variable if a was declared elsewhere
```

短变量声明只能出现在函数内部。在某些情况下，例如 "if"、"for "或 "switch "语句的初始化器，它们可以用来声明本地临时变量。

> 备注：详见 “重新声明” 一节