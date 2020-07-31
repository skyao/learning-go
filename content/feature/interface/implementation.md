---
date: 2020-07-04T23:50:00+08:00
title: 接口实现
weight: 862
menu:
  main:
    parent: "feature-interface"
description : "go语言的接口实现"
---



## go语言实战

### 隐式接口

类型通过实现那些方法来实现接口。 没有显式声明的必要；所以也就没有关键字“implements“。

隐式接口解藕了实现接口的包和定义接口的包：互不依赖。

因此，也就无需在每一个实现上增加新的接口名称。

常见的接口：

1.  [`fmt`](https://golang.org/pkg/fmt/) 包中定义的 [`Stringer`](https://golang.org/pkg/fmt/#Stringer)

   ```go
   type Stringer interface {
       String() string
   }
   ```

   `Stringer` 是一个可以用字符串描述自己的类型。`fmt`包 （还有许多其他包）使用这个来进行输出。

   ```go
   type Person struct {
   	Name string
   	Age  int
   }

   func (p Person) String() string {
   	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
   }
   ```



## Effective Go

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




