---
date: 2020-07-04T23:50:00+08:00
title: Type Switch
weight: 864
menu:
  main:
    parent: "feature-interface"
description : "go语言的Type Switch"
---

> type switch语句的语法详情见：[switch 语句](../../grammar/flow/switch.html)

## Effective Go

### 转换

https://golang.org/doc/effective_go.html#conversions

Sequence的String方法重现了Sprint已经为slices做的工作。(它的复杂度也是O(N²)，这很差。) 如果我们在调用Sprint之前将Sequence转换为一个普通的[]int，我们就可以分担这项工作（也可以加快它的速度）。

```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

这个方法是另一个从String方法安全调用Sprintf的转换技术的例子。因为两个类型（Sequence和[]int）是一样的，如果我们忽略类型名，那么在它们之间进行转换是合法的。这个转换并没有创建一个新的值，它只是暂时把现有的值当作一个新的类型。(还有其他合法的转换，比如从整数到浮点，确实会创建一个新的值。)

这是Go程序中的一个习惯做法，用来转换表达式的类型以访问不同的方法集。举个例子，我们可以使用现有的 sort.IntSlice 类型，将整个例子简化为这样。

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

现在，我们不再让Sequence实现多个接口（排序和打印），而是利用一个数据项转换为多种类型（Sequence、sort.IntSlice和[]int）的能力，每种类型都能完成一部分工作。这在实践中比较少见，但可以很有效。

### 接口转换和类型断言

https://golang.org/doc/effective_go.html#interface_conversions

type switch 是一种转换形式：它们接受一个接口，并在某种意义上，对于switch中的每一个case，将其转换为该case的类型。下面是fmt.Printf下的代码如何使用类型转换将一个值变成一个字符串的简化版本。如果它已经是一个字符串，我们要的是接口所持有的实际字符串值，而如果它有一个String方法，我们要的是调用该方法的结果。

```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

第一个case是找到一个具体的值；第二个case是将接口转换成另一个接口。这样混合类型是完全可以的。

如果我们只关心一种类型呢？如果我们知道这个值持有一个字符串，而我们只想提取它？一个one-case类型switch就可以了，但类型断言（type assertion）也可以。类型断言接受一个接口值，并从中提取一个指定显式类型的值。语法借鉴了打开类型转换的子句，但用的是显式类型而不是类型关键字。

```go
value.(typeName)
```

结果是一个静态类型typeName的新值。该类型必须是接口所持有的具体类型，或者是该值可以转换为的第二个接口类型。为了提取我们知道的值中的字符串，我们可以写。

```go
str := value.(string)
```

但如果发现值不包含字符串，程序就会因运行时错误而崩溃。为了防止这种情况，可以使用 "comma, ok" 这个习惯用法来安全地测试值是否是字符串。

```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

如果类型断言失败，str仍将存在，并且类型为string，但它的值为零，是一个空字符串。

为了说明这种能力，这里有一个if-else语句，相当于本节开头的类型切换。

```go
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

