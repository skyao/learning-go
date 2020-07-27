---
date: 2020-07-04T23:50:00+08:00
title: 数组类型
weight: 474
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的数组类型"
---

## 数组

> 摘录自 go语言实战

类型 `[n]T` 是一个数组，有 `n` 个类型为 `T` 的值。

数组定义访问如下，需要指定类型和数组大小：

```go
var a [10]int
```

注意：数组的长度是其类型的一部分，因此**数组不能改变大小**。

也可以在定义时直接创建数组，数组大小的设置可以有多种方式：

```go
a := [2]string{"a", "b"}
a := []string{"a", "b"}
a := [...]string{"a", "b"}
```

通过下标访问单个元素：

```go
var a [2]string
a[0] = "Hello"
a[1] = "World"
fmt.Println(a[0], a[1])
fmt.Println(a)
```

## Array types

https://golang.org/ref/spec#Array_types

数组是一个单一类型的元素的计数序列，称为元素类型。元素的数量称为数组的长度，绝不是负数。

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

长度是数组类型的一部分；它必须计算为一个非负常数，用int类型的值表示。数组a的长度可以通过内置函数len来计算。元素可以用0到len(a)-1的整数索引来寻址。数组类型总是一维的，但可以组成多维类型。

```go
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

## Arrays

https://golang.org/doc/effective_go.html#arrays

在规划内存的详细布局时，数组很有用，有时可以帮助避免分配，但主要是它们是切片的构建模块，也就是下一节的主题。为了给这个主题打下基础，下面说说关于数组的一些情况。

在Go和C中，数组的工作方式有很大的不同，在Go中：

- 数组就是值。将一个数组赋值给另一个数组会复制所有的元素。
- 特别是，如果你把一个数组传递给一个函数，它将收到一个数组的副本，而不是一个指向它的指针。
- 数组的大小是其类型的一部分。类型 [10]int 和 [20]int 是不同的。

值属性可能很有用，但也很昂贵；如果你想要类似C的行为和效率，你可以传递一个指针给数组。

```go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```

但即使这种风格也不是 go 的习惯用法。请用分片（slice）代替。



### 参考资料

- https://gobyexample-cn.github.io/arrays

