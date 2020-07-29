---
date: 2020-07-04T23:50:00+08:00
title: 方法
weight: 813
menu:
  main:
    parent: "feature-function"
description : "go语言的方法"
---

一般语言中，比如c/c++/Java/python等，函数和方法是等同的。但是在Go语言中，函数和方法有明确的区分，是两个不同的概念：

- 函数/Function：不属于任何结构体、类型，没有接收者

	- ```go
		func Add(a, b int) int {
			return a + b
		}
		```

- 方法/Method：和特定的结构体、类型关联，带有接收者

	- ```go
		type person struct {
			name string
		}
		
		func (p person) String() string{
			return "the person name is "+p.name
		}
		```

简单总结：

> Remember: a method is just a function with a receiver argument.
>
> 请记住：方法只是一个带有接收者参数的函数。

### 摘要：Methods vs Functions in Golang

https://medium.com/@ishagirdhar/methods-vs-functions-in-golang-c60586bfa6b4

方法还是函数？函数还是方法？

对于从Java或者其他面向对象的语言背景过来的人看来，第一直觉是处处都用结构体（struct）和方法（method），因为对象的行为总是由方法定义的。但在Golang中，我们既有函数又有方法，这种做法正确吗？

哪些地方我们需要使用方法，哪些地方我们需要函数？

我们先来看看Golang中什么是函数，什么是方法。

**函数（function）**接受一些参数作为输入，并产生一些输出。对于相同的输入，函数总是会产生相同的输出。这意味着它不依赖于状态。类型是作为函数的参数传递的。

Go中的**方法（Method）**是带有特定 receiver(type) 参数上的函数。它定义了类型的行为，它应该使用类型的状态。

但是，如果我们在结构体里面没有状态，那么我们是不是根本就不要定义方法呢？答案是我们可以定义，但这主要是为了对该特定类型的方法进行逻辑分组。

不定义方法的规则是，如果

- 不需要依赖状态
- 可以在任何实现特定接口的类型上执行这个函数，这意味着不需要限制这个函数属于某个特定的类型。

### 接收者的按值传递

> 摘录自 [Go语言实战笔记（八）| Go 函数方法](https://www.flysnow.org/2017/03/31/go-in-action-go-method.html)

Go语言里有两种类型的接收者：值接收者和指针接收者。

- 使用值接收者: 在调用的时候，方法使用的其实是值接收者的一个副本，所以对该值的任何操作，不会影响原来的值接收者（或者说类型变量）。

- 使用指针接收者：因为指针接收者传递的是一个指向原值指针的副本（即指针的副本），其指向的还是原来类型的值，所以修改时，同时也会影响原来值接收者（类型变量）的值。

总结：

在调用方法的时候，传递的接收者本质上都是副本，只不过可以是值的副本，也可以是指向这个值的指针的副本。指针具有指向原有值的特性，所以修改了指针指向的值，也就修改了原有的值。

### Effective Go

Pointers vs. Values 

https://golang.org/doc/effective_go.html#pointers_vs_values

正如我们在ByteSize中看到的那样，可以为任何命名类型（除了指针或接口）定义方法；接收者不一定是结构体。

在上面关于切片的讨论中，我们写了一个Append函数。我们可以把它定义为一个关于切片的方法。要做到这一点，我们首先声明一个可以绑定方法的命名类型，然后使该方法的接收者是该类型的值。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```

这仍然需要该方法返回更新后的切片。我们可以通过重新定义该方法来消除这种笨拙，将一个指向ByteSlice的指针作为它的接收器，这样该方法就可以覆盖调用者的切片。

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

事实上，我们还可以做得更好。如果我们修改函数，使它看起来像一个标准的写方法，像这样。

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

那么*ByteSlice类型满足标准接口io.Writer，这很方便。例如，我们可以打印成一个。

```go
    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

 关于接收者的指针与值的规则是：值方法可以在指针和值上被调用，但指针方法只能在指针上被调用。

这个规则的产生是因为指针方法可以修改接收者；在值上调用它们会导致该方法接收到一个值的副本，所以任何修改都会被丢弃。因此，语言不允许这种错误。不过有一个方便的例外。当值是可寻址的时候，语言通过自动插入地址操作符来处理在值上调用指针方法的常见情况。在我们的例子中，变量b是可寻址的，所以我们可以只用b.Write来调用它的Write方法。编译器会帮我们改写成 (&b).Write。

顺便说一下，在字节切片上使用Write的想法是实现bytes.Buffer的核心。

### 摘要：方法的本质

> 摘录自 [函数——go世界中的一等公民](https://segmentfault.com/a/1190000023340324) “方法的本质” 一节

go的方法就是语法糖：实际上Method就是将receiver作为函数的第一个参数输入的语法糖而已，本质上和函数没有区别



### 参考资料

- [函数、方法和接口](https://chai2010.cn/advanced-go-programming-book/ch1-basic/ch1-04-func-method-interface.html)
-  [函数——go世界中的一等公民](https://segmentfault.com/a/1190000023340324)