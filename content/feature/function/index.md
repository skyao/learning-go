---
date: 2020-07-04T23:50:00+08:00
title: 函数
weight: 810
description : "介绍Golang的函数"
---



在go的世界中，函数是一等公民，可以给变量赋值，可以作为参数传递，也可以直接赋值。

### 多值返回

https://golang.org/doc/effective_go.html#multiple-returns

Go的一个不同寻常的特点是，**函数和方法可以返回多个值**。这种形式可以用来改进C程序中的几个笨拙的习语：带内错误返回，如-1代表EOF和修改按地址传递的参数。

在C语言中，写错误的信号是一个负数，错误代码被秘密存放在一个易失性的位置。在Go中，Write可以返回计数（count）和错误（error）。"是的，你写了一些字节，但不是全部，因为你填满了设备". 来自包os的文件上的Write方法的签名是。

```go
func (file *File) Write(b []byte) (n int, err error)
```

和文档中说的一样，当 n != len(b) 时，它返回写入的字节数和一个非nil错误。这是一种常见的风格；更多的例子请参见错误处理一节。

类似的方法避免了传递指针到返回值以模拟引用参数的需要。下面是一个简单的函数，用于从字节片中的某个位置抓取一个数字，返回数字和下一个位置。

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

你可以用它来扫描输入切片b中的数字，像这样。

```go
    for i := 0; i < len(b); {
        x, i = nextInt(b, i)
        fmt.Println(x)
    }
```


### 命名结果参数

Go函数的返回或结果 "参数 "可以被赋予名称，并作为常规变量使用，就像传入参数一样。当命名时，它们在函数开始时被初始化为其类型的零值；如果函数执行一个没有参数的返回语句，结果参数的当前值被用作返回值。

这些名称并不是强制性的，但它们可以使代码更短、更清晰：它们是文档。如果我们给 nextInt 的结果命名，就会很明显地知道哪个返回的 int 是哪个。

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

因为被命名的结果是初始化的，并且与一个不加修饰的返回相联系，所以它们可以简化以及澄清。下面是一个很好地使用它们的 io.ReadFull 版本。

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### defer

详见 defer 语句。



### 参考资料

- [函数——go世界中的一等公民](https://segmentfault.com/a/1190000023340324)