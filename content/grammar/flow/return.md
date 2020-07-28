---
date: 2020-07-04T23:50:00+08:00
title: return语句
weight: 606
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 return 语句"
---

## golang语言规范

https://golang.org/ref/spec#Return_statements

函数F中的 "return"语句会终止F的执行，并可选地提供一个或多个结果值。在F返回给调用者之前，任何被F deferred 的函数都会被执行。

```
ReturnStmt = "return" [ ExpressionList ] .
```

在没有结果类型的函数中，"return"语句不能指定任何结果值。

```go
func noResult() {
	return
}
```

有三种方法可以从一个具有结果类型的函数中返回值：

1. 返回值可以在 "return"语句中明确列出。每个表达式必须是单值的，并且可以分配给函数结果类型的相应元素。

	```go
	func simpleF() int {
		return 2
	}
	
	func complexF1() (re float64, im float64) {
		return -7.0, -4.0
	}
	```

2. "return"语句中的表达式列表可以是对一个多值函数的单次调用。其效果就好比该函数返回的每个值都被分配到一个临时变量中，其类型为相应的值，然后用 "return"语句列出这些变量，这时就适用前一种情况的规则。

	```go
	func complexF2() (re float64, im float64) {
		return complexF1()
	}
	```

3. 如果函数的结果类型为其结果参数指定了名称，那么表达式列表可以为空。结果参数作为普通的局部变量，函数可以根据需要为它们赋值。return 语句返回这些变量的值。

	```go
	func complexF3() (re float64, im float64) {
		re = 7.0
		im = 4.0
		return
	}
	
	func (devnull) Write(p []byte) (n int, _ error) {
		n = len(p)
		return
	}
	```

无论如何声明，所有的结果值在进入函数时都初始化为其类型的零值。指定结果的 "return" 语句会在执行任何deferred 函数之前设置结果参数。

实现限制：如果在返回的地方有不同的实体（常量、类型或变量）与结果参数同名，编译器可以不允许在 "return" 语句中使用空表达式列表。

