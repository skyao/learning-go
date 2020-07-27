---
date: 2020-07-04T23:50:00+08:00
title: iota
weight: 453
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的iota"
---

### iota

https://golang.org/ref/spec#Iota

在常量声明中，预先声明的标识符 iota 代表连续的非类型整数常量。它的值是该常量声明中各自ConstSpec的索引，从0开始。它可以用来构造一组相关的常量。

```go
const (
	c0 = iota  // c0 == 0
	c1 = iota  // c1 == 1
	c2 = iota  // c2 == 2
)

const (
	a = 1 << iota  // a == 1  (iota == 0)
	b = 1 << iota  // b == 2  (iota == 1)
	c = 3          // c == 3  (iota == 2, unused)
	d = 1 << iota  // d == 8  (iota == 3)
)

const (
	u         = iota * 42  // u == 0     (untyped integer constant)
	v float64 = iota * 42  // v == 42.0  (float64 constant)
	w         = iota * 42  // w == 84    (untyped integer constant)
)

const x = iota  // x == 0
const y = iota  // y == 0
```

根据定义，iota在同一个ConstSpec中的多次使用都具有相同的值：

```go
const (
	bit0, mask0 = 1 << iota, 1<<iota - 1  // bit0 == 1, mask0 == 0  (iota == 0)
	bit1, mask1                           // bit1 == 2, mask1 == 1  (iota == 1)
	_, _                                  //                        (iota == 2, unused)
	bit3, mask3                           // bit3 == 8, mask3 == 7  (iota == 3)
)
```

最后一个例子利用了最后一个非空表达式列表的隐式重复。

