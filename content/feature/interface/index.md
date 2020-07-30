---
date: 2020-07-04T23:50:00+08:00
title: 接口
weight: 860
description : "介绍Golang的内建函数"
---



### Interfaces

https://golang.org/doc/effective_go.html#interfaces

Go中的接口提供了一种指定对象行为的方法：如果某个东西可以做这个，那么它就可以在这里使用。我们已经看到了几个简单的例子；自定义的打印机可以通过一个String方法实现，而Fprintf可以通过一个Write方法向任何东西生成输出。只有一个或两个方法的接口在Go代码中很常见，通常会被赋予一个由方法派生的名称，比如io.Writer用于实现Write方法的东西。

一个类型可以实现多个接口。例如，一个集合如果实现了 sort.Interface，就可以通过包 sort 中的例程进行排序，其中包含 Len()、Less(i, j int) bool 和 Swap(i, j int)，它还可以有一个自定义的格式器。在这个人为的例子中，Sequence同时满足这两个条件。

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

### 接口命名

https://golang.org/doc/effective_go.html#interface-names

按照惯例，单方法接口的命名是由方法名加上 -er 后缀或类似的修饰来构造一个代理名词：Reader, Writer, Formatter, CloseNotifier等等。

这样的名字有很多，尊重它们和它们所使用的函数名是很有成效的。Read、Write、Close、Flush、String等都有规范的标志和含义。为了避免混淆，不要给你的方法起这些名字，除非它有相同的签名和含义。反过来说，如果你的类型实现了一个与一个著名类型上的方法具有相同含义的方法，就给它相同的名称和签名；调用你的字符串转换方法String而不是ToString。

### 接口检查

https://golang.org/doc/effective_go.html#blank_implements

正如我们在上面关于接口的讨论中所看到的，类型不需要明确声明它实现了接口。相反，类型只要实现了接口的方法就实现了接口。在实践中，大多数接口转换都是静态的，因此在编译时进行检查。例如，将一个 *os.File 传给一个期望有 io.Reader 的函数，除非 *os.File 实现了 io.Reader 接口，否则不会被编译。

但有些接口检查确实是在运行时进行的。其中一个例子是在 encoding/json 包中，它定义了一个 Marshaler 接口。当JSON编码器接收到一个实现该接口的值时，编码器会调用该值的 marshaling 方法将其转换为JSON，而不是进行标准转换。编码器在运行时用一个类型断言检查这个属性，比如。

```go
m, ok := val.(json.Marshaler)
```

如果只需要询问一个类型是否实现了一个接口，而不实际使用接口本身，或许作为错误检查的一部分，使用空白标识符来忽略类型假定的值。

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

出现这种情况的一个地方是，当需要在实现类型的包内保证它确实满足接口的时候。如果一个类型--例如，json.RawMessag--需要自定义的JSON表示，它应该实现 json.Marshaler，但没有静态转换会导致编译器自动验证这一点。如果该类型无意中没有满足接口，JSON编码器仍然会工作，但不会使用自定义的实现。为了保证实现的正确性，可以在包中使用空白标识符的全局声明。

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

在这个声明中，涉及到将*RawMessage转换为Marshaler的赋值，需要*RawMessage实现Marshaler，并且在编译时将检查该属性。如果json.Marshaler接口发生变化，这个包将不再编译，我们会被通知需要更新。

在这个构造中出现空白标识符，说明声明的存在只是为了类型检查，而不是为了创建一个变量。不过不要对每个满足接口的类型都这样做。按照惯例，这种声明只有在代码中已经没有静态转换时才会使用，这是很罕见的事件。