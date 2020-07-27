---
date: 2020-07-04T23:50:00+08:00
title: 数字类型
weight: 472
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的数字类型"
---

### Numeric types

https://golang.org/ref/spec#Numeric_types

数值类型表示整数或浮点值的集合。预先声明的与架构无关的数值类型有：

```go
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32
```

一个n位整数的值是n位宽，用二补码算术（[two's complement arithmetic](https://en.wikipedia.org/wiki/Two's_complement)）表示。

还有一组预先声明的数值类型，其大小与实现有关。

```go
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value
```

为了避免可移植性问题，所有的数值类型都是已定义类型，因此除了byte（uint8的别名）和rune（int32的别名）之外，其他类型都是不同的。当在表达式或赋值中混合使用不同的数值类型时，需要进行显式转换。例如，int32和int不是同一类型，即使它们在特定架构上可能具有相同的大小。









