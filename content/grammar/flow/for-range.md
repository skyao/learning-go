---
date: 2020-07-04T23:50:00+08:00
title: for-range语句
weight: 604
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 for-rang 语句"
---



## go语言规范

### For statements with `range` clause

https://golang.org/ref/spec#For_statements

带有 "range"子句的 "for" 语句会遍历数组、切片、字符串或map的所有条目，或通道（channel）上接收的值。对于每个条目，如果存在，它将迭代值分配给相应的迭代变量，然后执行该块。

```
RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range" Expression .
```

"range"子句中右边的表达式称为range表达式，它可以是数组、数组指针、分片、字符串、map或允许接收操作的通道。与赋值一样，如果存在，左边的操作数必须是可寻址或映射索引表达式；它们表示迭代变量。如果范围表达式是通道，最多允许一个迭代变量，否则最多可以有两个。如果最后一个迭代变量是空白标识符，那么范围子句就相当于没有该标识符的同一个子句。

范围表达式x在开始循环之前被评估一次，但有一个例外：如果最多存在一个迭代变量，并且len(x)是常数，则不评估范围表达式。

左边的函数调用在每次迭代时都会被评估一次。对于每一次迭代，如果存在各自的迭代变量，就会产生如下的迭代值。

```go
Range expression                          1st value          2nd value

array or slice  a  [n]E, *[n]E, or []E    index    i  int    a[i]       E
string          s  string type            index    i  int    see below  rune
map             m  map[K]V                key      k  K      m[k]       V
channel         c  chan E, <-chan E       element  e  E
```

1. 对于数组、数组指针或切片值a，从元素索引0开始，按递增顺序产生索引迭代值。如果最多存在一个迭代变量，则 range 循环产生从0到 `len(a)-1` 的迭代值，并且不对数组或分片本身进行索引。对于nil slice，迭代次数为0. 对于字符串值，"range循环"会产生从 0 到 `len(a)-1` 的迭代值，并且不对数组或分片本身进行索引。

2. 对于字符串值，"range"子句从字节索引 0 开始对字符串中的Unicode code point 进行迭代。在连续迭代时，索引值将是字符串中连续的UTF-8编码 code point 的第一个字节的索引，第二个值，类型为 rune，将是对应code point的值。如果迭代遇到无效的UTF-8序列，第二个值将是`0xFFFD`，即Unicode replacement(替换)字符，下一次迭代将推进字符串中的一个字节。

3. 对map的迭代顺序没有指定，也不能保证每次迭代的顺序相同。如果在迭代过程中删除了一个尚未到达的map条目，将不会产生相应的迭代值。如果在迭代过程中创建了一个map条目，该条目可能在迭代过程中产生，也可能被跳过。对于每个创建的条目，以及从一个迭代到下一个迭代，选择可能不同。如果map为nil，则迭代次数为0。

4. 对于通道，产生的迭代值是通道上连续发送的值，直到通道关闭。如果通道为nil，则range表达式永远阻塞。

迭代值像赋值语句一样被赋值到相应的迭代变量中。

迭代变量可以由 "range "子句使用短变量声明的形式（:=）来声明。在这种情况下，它们的类型被设置为各自迭代值的类型，它们的作用域是 "for"语句的块；它们在每次迭代中被重复使用。如果迭代变量是在 "for"语句之外声明的，那么在执行后它们的值将是最后一次迭代的值。

```go
var testdata *struct {
	a *[7]int
}
for i, _ := range testdata.a {
	// testdata.a is never evaluated; len(testdata.a) is constant
	// i ranges from 0 to 6
	f(i)
}

var a [10]string
for i, s := range a {
	// type of i is int
	// type of s is string
	// s == a[i]
	g(i, s)
}

var key string
var val interface{}  // element type of m is assignable to val
m := map[string]int{"mon":0, "tue":1, "wed":2, "thu":3, "fri":4, "sat":5, "sun":6}
for key, val = range m {
	h(key, val)
}
// key == last map key encountered in iteration
// val == map[key]

var ch chan Work = producer()
for w := range ch {
	doWork(w)
}

// empty a channel
for range ch {}
```

## Effective Go

https://golang.org/doc/effective_go.html#for

如果你正在循环数组、切片、字符串或map，或者从一个通道读取，range子句可以管理循环。

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

如果你只需要range内的第一项（键或索引），就放弃第二项。

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

如果你只需要范围中的第二项（值），使用空白的标识符（下划线）来丢弃第一项。

```go
sum := 0
for _, value := range array {
    sum += value
}
```

对于字符串，range 为你做了更多的工作，通过解析UTF-8分解出各个Unicode code point。错误的编码会消耗一个字节，并产生替换符U+FFFD。(名称(与相关的内置类型) rune是单个Unicode code point的go术语。详情请参见语言规范。) 。循环

```go
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```

打印：

```go
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```



### 参考资料

- [Dig101: Go 之 for-range 排坑指南](https://gocn.vip/topics/9573)