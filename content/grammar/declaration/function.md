---
date: 2020-07-04T23:50:00+08:00
title: 函数声明
weight: 456
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的函数声明"
---

### Function declaration

https://golang.org/ref/spec#Function_declarations

函数声明将标识符，即函数名，与函数绑定。

```
FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = Block .
```

如果函数的签名声明了结果参数，那么函数体的语句列表必须以终止语句结束。

```go
func IndexRune(s string, r rune) int {
	for i, c := range s {
		if c == r {
			return i
		}
	}
	// invalid: missing return statement
}
```

函数声明可以省略主体。这样的声明提供了在Go之外实现的函数的签名，例如汇编例程。

```go
func min(x int, y int) int {
	if x < y {
		return x
	}
	return y
}

func flushICache(begin, end uintptr)  // implemented externally
```

