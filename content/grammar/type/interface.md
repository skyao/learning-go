---
date: 2020-07-04T23:50:00+08:00
title: 接口类型
weight: 479
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的接口类型"
---

## Interface types

https://golang.org/ref/spec#Interface_types

接口类型指定了一个称为 interface/接口的方法集。接口类型的变量可以存储任何类型的值，其方法集是接口的任何超集。这样的类型被称为**实现**了接口。未初始化的接口类型变量的值是nil。

```
InterfaceType      = "interface" "{" { ( MethodSpec | InterfaceTypeName ) ";" } "}" .
MethodSpec         = MethodName Signature .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

接口类型可以通过方法规范明确地指定方法，也可以通过接口类型名称嵌入其他接口的方法。

```go
// A simple File interface.
interface {
	Read([]byte) (int, error)
	Write([]byte) (int, error)
	Close() error
}
```

每个显式指定方法的名称必须是唯一的，不能是空白的。

```go
interface {
	String() string
	String() string  // illegal: String not unique
	_(x int)         // illegal: method must have non-blank name
}
```

可以有多个类型实现一个接口。例如，如果两个类型S1和S2的方法集为

```go
func (p T) Read(p []byte) (n int, err error)
func (p T) Write(p []byte) (n int, err error)
func (p T) Close() error
```

(其中T代表S1或S2)，那么File接口是由S1和S2同时实现的，不管S1和S2可能有什么其他方法或共享什么方法。

类型实现了由其方法的任何子集组成的任何接口，因此可以实现几个不同的接口。例如，所有类型都实现空接口。

```go
interface{}
```

类似地，考虑这个接口规范，它出现在一个类型声明中，定义了一个叫做Locker的接口。

```go
type Locker interface {
	Lock()
	Unlock()
}
```

如果S1和S2也实现：

```go
func (p T) Lock() { … }
func (p T) Unlock() { … }
```

它们实现了Locker接口和File接口。

接口T可以使用一个（可能是限定的）接口类型名E来代替方法规范。这就是所谓的在T中嵌入（*embedding*）接口E。 T的方法集是T的显式声明方法和T的嵌入式接口的方法集的联合。

```go
type Reader interface {
	Read(p []byte) (n int, err error)
	Close() error
}

type Writer interface {
	Write(p []byte) (n int, err error)
	Close() error
}

// ReadWriter's methods are Read, Write, and Close.
type ReadWriter interface {
	Reader  // includes methods of Reader in ReadWriter's method set
	Writer  // includes methods of Writer in ReadWriter's method set
}
```

方法集的联合体包含了每个方法集的（导出的和未导出的）方法，每个方法集只有一次，而且名称相同的方法必须有相同的签名。

```go
type ReadCloser interface {
	Reader   // includes methods of Reader in ReadCloser's method set
	Close()  // illegal: signatures of Reader.Close and Close are different
}
```

接口类型T不得将自己或任何嵌入T的接口类型，递归地嵌入。

```go
// illegal: Bad cannot embed itself
type Bad interface {
	Bad
}

// illegal: Bad1 cannot embed itself using Bad2
type Bad1 interface {
	Bad2
}
type Bad2 interface {
	Bad1
}
```

## Interface

https://golang.org/doc/effective_go.html#interfaces

Go中的接口提供了一种指定对象行为的方法：如果某个东西可以做这个，那么它就可以在这里使用。我们已经看到了几个简单的例子；自定义的打印机可以通过一个String方法实现，而Fprintf可以通过一个Write方法向任何东西生成输出。只有一个或两个方法的接口在Go代码中很常见，通常会被赋予一个由方法派生的名称，比如io.Writer用于实现Write方法的东西。

类型可以实现多个接口。例如，一个集合如果实现了 sort.Interface，就可以通过包 sort 中的例程进行排序，其中包含 Len()、Less(i, j int) bool 和 Swap(i, j int)，它还可以有一个自定义的格式器。在这个人为的例子中，Sequence同时满足这两个条件。

```go
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Copy returns a copy of the Sequence.
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    s = s.Copy() // Make a copy; don't overwrite argument.
    sort.Sort(s)
    str := "["
    for i, elem := range s { // Loop is O(N²); will fix that in next example.
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```