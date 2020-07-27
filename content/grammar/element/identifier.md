---
date: 2020-07-04T23:50:00+08:00
title: 标识符
weight: 413
menu:
  main:
    parent: "grammar-element"
description : "go语言中的标识符"
---

> 备注：摘录自 golang语言规范 https://golang.org/ref/spec#Blank_identifier

### 空白标识符

空白标识符由下划线字符 `_` 表示。它作为匿名的占位符，而不是常规的（非空白）标识符，在声明、操作数和赋值中具有特殊意义。

### 预定义的标识符

universe block（宇宙块）中隐式声明了以下标识符：

```go
Types:
	bool byte complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr

Constants:
	true false iota

Zero value:
	nil

Functions:
	append cap close complex copy delete imag len
	make new panic print println real recover
```

### 导出的标识符

标识符可以导出，以便从另一个包中访问它。标识符在以下两种情况下被导出：

1. 标识符名称的第一个字符是Unicode大写字母(Unicode class "Lu")；和
2. 标识符是在package block（包块）中声明的，或者是字段名或方法名。

所有其他的标识符都不会被导出。

### 标识符的唯一性

给定一组标识符，如果一个标识符与集合中的其他标识符不同，则称为唯一标识符。如果两个标识符的拼写不同，或者它们出现在不同的包中并且没有被导出，那么它们就是不同的。否则，它们是相同的。