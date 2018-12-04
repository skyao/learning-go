---
date: 2018-12-04T23:50:00+08:00
title: 变量
weight: 414
menu:
  main:
    parent: "grammar-basic"
description : "go语言中的变量"
--- 

### 变量定义

`var` 语句定义一个变量的列表；跟函数的参数列表一样，类型在后面。

`var` 语句可以定义在包或函数级别。

```go
var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```

### 变量初始化

```go
// 变量定义可以包含初始值，每个变量对应一个。
var i, j int = 1, 2

func main() {
    // 如果初始化是使用表达式，则可以省略类型；
    // 变量从初始值中获得类型。
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```

也可以用这个方式初始化多个变量，每个变量一行代码：

```go
var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)
```

如果变量在定义时没有明确的初始化，则会赋值为该类型对应的**零值**：

- 数值类型为 `0`，
- 布尔类型为 `false`，
- 字符串为 `""`（空字符串）

## 短声明变量

在函数中，`:=` 简洁赋值语句在明确类型的地方，可以用于替代 `var` 定义。

```go
func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"
}
```

函数外的每个语句都必须以关键字开始（`var`、`func`、等等），`:=` 结构不能使用在函数外。

### 多赋值模式

上面同时赋值多个变量的模式，可以用于if, for 等语句，实现在if, for 语句中的多赋值模式：

```go
for i, j, s := 0, 5, "a"; i < 3 && j < 100 && s != "aaaaa"; i, j, s = i+1, j+1, s + "a"  {
    fmt.Println("Value of i, j, s:", i, j, s)
}
```

