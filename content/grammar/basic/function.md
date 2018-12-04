---
date: 2018-12-04T23:50:00+08:00
title: 函数
weight: 413
menu:
  main:
    parent: "grammar-basic"
description : "go语言中的函数"
---

### 语法

Golang的函数语法和Java、C等有非常大的不同：

- 函数参数定义是类型在变量名之后，而不是变量名之前
- 函数返回值定义在函数签名之后，而不是函数签名之前
- 函数可以返回多个值

```go
func swap(x, y string) (string, string) {
	return y, x
}
```

### 参数

函数可以没有参数或有多个参数。

注意类型在变量名 **之后**。

```go
func add(x int, y int) int {
	return x + y
}
```

当两个或多个连续的参数是同一类型，则除了最后一个类型之外，其他都可以省略。

```go
func add(x, y int) int {
	return x + y
}
```

### 返回值

函数可以返回任意数量的返回值。这点和很多语言不同：

```go
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

Go 的返回值可以被命名，并且像变量那样使用。

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

