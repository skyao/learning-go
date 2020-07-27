---
date: 2020-07-04T23:50:00+08:00
title: 类型
weight: 470
description : "介绍Go语言的类型系统"
---

## Type

https://golang.org/ref/spec#Types

类型决定了一组值，以及对这些值的特定操作和方法。如果有类型名称，可以用类型名称来表示，也可以用类型字面量来指定，由现有的类型组成一个类型。

```
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
	    SliceType | MapType | ChannelType .
```

语言预先声明某些类型名称。其他类型是通过类型声明引入的。复合类型--数组、结构体、指针、函数、接口、切片、映射和通道类型--可以使用类型字面量来构造。

每个类型T都有一个底层类型（*underlying type*）。如果T是预定义的布尔类型、数值类型、字符串类型之一，或者是类型字面量，那么对应的底层类型就是T本身。否则，T的底层类型是T在其类型声明中引用的类型的底层类型。

```go
type (
	A1 = string
	A2 = A1
)

type (
	B1 string
	B2 B1
	B3 []B1
	B4 B3
)
```

string、A1、A2、B1、B2的底层类型是string。[]B1、B3和B4的底层类型是[]B1。

### 方法集

类型可以有一个与之相关的方法集。接口类型的方法集就是它的接口。任何其他类型T的方法集由所有用接收者类型T声明的方法组成。相应的指针类型`*T`的方法集是用接收者 `*T`或`T`声明的所有方法的集合（也就是说，它也包含T的方法集）。更多的规则适用于包含嵌入式字段的结构体，如结构体类型一节所述。任何其他类型的方法集都是空的。在方法集中，每个方法必须有一个唯一的非空方法名。

类型的方法集决定了该类型实现的接口和可以使用该类型的接收者调用的方法。

## 类型和值的属性

https://golang.org/ref/spec#Properties_of_types_and_values

### 类型一致性

两个类型要么相同，要么不同。

已定义类型（defined type）总是不同于其他类型。否则，如果它们的底层类型字面量在结构上是等价的，那么两个类型就是相同的；也就是说，它们具有相同的字面量结构，相应的组件具有相同的类型。详言之。

- 如果两个数组类型具有相同的元素类型和相同的数组长度，那么它们就是相同的。
- 如果两个分片类型具有相同的元素类型，则它们是相同的。
- 如果两个结构体类型具有相同的字段序列，并且对应的字段具有相同的名称、相同的类型和相同的标签，则两个结构体类型是相同的。来自不同包的非导出字段名总是不同的。
- 如果两个指针类型具有相同的基础类型，那么它们就是相同的。
- 如果两个函数类型具有相同数量的参数和结果值，对应的参数和结果类型相同，并且两个函数都是变量，或者都不是变量，那么这两个函数类型就是相同的。**参数和结果名称不需要匹配**。
- 如果两个接口类型具有相同的方法集，名称相同，函数类型相同，则两个接口类型是相同的。来自不同包的非导出方法名总是不同的。**方法的顺序无关紧要**。
- 如果两个映射类型有相同的键和元素类型，那么它们就是相同的。
- 如果两个通道类型具有相同的元素类型和相同的方向，那么它们就是相同的。

给定声明：

```go
type (
	A0 = []string
	A1 = A0
	A2 = struct{ a, b int }
	A3 = int
	A4 = func(A3, float64) *A0
	A5 = func(x int, _ float64) *[]string
)

type (
	B0 A0
	B1 []string
	B2 struct{ a, b int }
	B3 struct{ a, c int }
	B4 func(int, float64) *B0
	B5 func(x int, y float64) *A1
)

type	C0 = B0
```

这些类型是相同的：

```go
A0, A1, and []string
A2 and struct{ a, b int }
A3 and int
A4, func(int, float64) *[]string, and A5

B0 and C0
[]int and []int
struct{ a, b *T5 } and struct{ a, b *T5 }
func(x int, y float64) *[]string, func(int, float64) (result *[]string), and A5
```

B0和B1是不同的，因为它们是由不同的类型定义创建的新类型；func(int，float64) *B0和func(x int，y float64) *[]string是不同的，因为B0与[]string不同。

### 可分配性

如果以下条件之一适用，则值x可分配给类型T的变量（"`x` is assignable to `T`"）。

- x的类型与T相同。
- x的类型V和T的底层类型相同，并且V或T中至少有一个不是定义类型。
- T是一个接口类型，x实现了T。
- x是双向通道值，T是通道类型，x的类型V和T具有相同的元素类型，V或T中至少有一个不是定义类型。
- x是预先声明的标识符nil，T是指针、函数、片、映射、通道或接口类型。
- x是一个可由T类型的值表示的非类型常量。

### 可表示性

如果下列条件之一适用，则常数x可由类型为T的值表示：

- x在由T决定的值集合中。
- T是一个浮点类型，并且x可以被四舍五入到T的精度而不会溢出。四舍五入使用IEEE 754四舍五入到偶数的规则，但将IEEE负零进一步简化为无符号零。注意，常量值永远不会产生IEEE负零、NaN或无穷大。
- T是复数类型，x的分量real(x)和 imag(x)可以用T的分量类型(float32或float64)的值表示。



```go
x                   T           x is representable by a value of T because

'a'                 byte        97 is in the set of byte values
97                  rune        rune is an alias for int32, and 97 is in the set of 32-bit integers
"foo"               string      "foo" is in the set of string values
1024                int16       1024 is in the set of 16-bit integers
42.0                byte        42 is in the set of unsigned 8-bit integers
1e10                uint64      10000000000 is in the set of unsigned 64-bit integers
2.718281828459045   float32     2.718281828459045 rounds to 2.7182817 which is in the set of float32 values
-1e-1000            float64     -1e-1000 rounds to IEEE -0.0 which is further simplified to 0.0
0i                  int         0 is an integer value
(42 + 0i)           float32     42.0 (with zero imaginary part) is in the set of float32 values
```



```go
x                   T           x is not representable by a value of T because

0                   bool        0 is not in the set of boolean values
'a'                 string      'a' is a rune, it is not in the set of string values
1024                byte        1024 is not in the set of unsigned 8-bit integers
-1                  uint16      -1 is not in the set of unsigned 16-bit integers
1.1                 int         1.1 is not an integer value
42i                 float32     (0 + 42i) is not in the set of float32 values
1e1000              float64     1e1000 overflows to IEEE +Inf after rounding
```

