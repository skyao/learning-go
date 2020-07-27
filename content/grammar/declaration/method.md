---
date: 2020-07-04T23:50:00+08:00
title: 方法声明
weight: 457
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的方法声明"
---

### Method declaration

https://golang.org/ref/spec#Method_declarations

方法是一个带有 *receiver* （接收者）的函数。方法声明将标识符，即方法名，绑定到一个方法上，并将该方法与接受者的基本类型联系起来。

```
MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
Receiver   = Parameters .
```

接收者是通过方法名前的额外参数部分指定的。这个参数部分必须声明一个非变量参数，即 receiver（接收者）。它的类型必须是一个 defined type（定义类型）T或者一个指向定义类型T的指针。T被称为接收者的基本类型。接收者基类类型不能是指针或接口类型，它必须与方法定义在同一个包中。该方法被称为绑定到它的接收者基本类型上，并且该方法名称只有在类型T或*T的选择器中才可见。

非空白的接收者标识符必须在方法签名中是唯一的。如果接收者的值没有在方法主体中引用，那么它的标识符可以在声明中省略。一般来说，这也适用于函数和方法的参数。

对于基础类型来说，与它绑定的方法的非空名称必须是唯一的。如果基础类型是结构体类型，则非空的方法和字段名必须是不同的。

给定定义类型Point，声明：

```go
func (p *Point) Length() float64 {
	return math.Sqrt(p.x * p.x + p.y * p.y)
}

func (p *Point) Scale(factor float64) {
	p.x *= factor
	p.y *= factor
}
```

将方法 Length 和 Scale（接收者类型为*Point）绑定到基本类型Point上。

方法的类型是以接收者为第一参数的函数类型。例如，方法Scale的类型是：

```go
func(p *Point, factor float64)
```

当然，这样声明的函数不是方法。

> 备注：可以参考文章  [函数——go世界中的一等公民](https://segmentfault.com/a/1190000023340324) 中的 “方法的本质” 一节
>
> “go里面其实方法就是语法糖，实际上Method就是将receiver作为函数的第一个参数输入的语法糖而已，本质上和函数没有区别”