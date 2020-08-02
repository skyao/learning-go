---
date: 2020-07-04T23:50:00+08:00
title: Panic
weight: 920
description : "介绍Golang的Panic机制"
---

## go by example

https://gobyexample-cn.github.io/panic

`panic` 意味着有些出乎意料的错误发生。 通常我们用它来表示程序正常运行中不应该出现的错误， 或者我们不准备优雅处理的错误。

我们将使用 panic 来检查这个站点上预期之外的错误。 而该站点上只有一个程序：触发 panic。

panic 的一种常见用法是：当函数返回我们不知道如何处理（或不想处理）的错误值时，中止操作。 如果创建新文件时遇到意外错误该如何处理？这里有一个很好的 `panic` 示例。

```go
package main

import "os"

func main() {

    panic("a problem")

    _, err := os.Create("/tmp/file")
    if err != nil {
        panic(err)
    }
}
```

运行程序将会导致 panic： 输出一个错误消息和协程追踪信息，并以非零的状态退出程序:

```bash
$ go run panic.go
panic: a problem
goroutine 1 [running]:
main.main()
    /.../panic.go:12 +0x47
...
exit status 2
```

注意，与某些使用 exception 处理错误的语言不同， 在 Go 中，通常会尽可能的使用返回值来标示错误。

## Effective Go

### panic

https://golang.org/doc/effective_go.html#panic

向调用者报告错误的通常方法是返回 error 作为一个额外的返回值。规范的 Read 方法是一个著名的实例；它返回一个字节数和一个错误。但如果错误无法恢复怎么办？有时，程序根本无法继续。

为此，有一个内置的函数panic，它实际上会产生一个运行时错误，使程序停止（但请看下一节）。这个函数接收一个任意类型的参数--通常是一个字符串--在程序死亡时打印出来。它也是一种指示不可能发生的事情的方法，比如退出一个无限循环。

```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

这只是一个例子，但真正的库函数应该避免 panic。如果问题可以被 recover 或解决，让事情继续运行总比把整个程序拆掉要好。一个可能的反例是在初始化过程中：如果库真的无法自我创建，那么可以说，panic 是合理的。

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

### Recover

https://golang.org/doc/effective_go.html#recover

当调用 panic 时，包括隐含的运行时错误，如索引分片出界或类型断言失败，它会立即停止执行当前函数，并开始解开（unwind） goroutine 的堆栈，沿途运行任何 defer 函数。如果该解卷（unwind）到达 goroutine 的栈顶，程序就会死亡。然而，可以使用内置函数 recover 来重新获得goroutine的控制权并恢复正常执行。

对 recover 的调用会停止解卷（unwind），并返回传递给 panic 的参数。因为在解卷（unwind）时只有在defer函数内部的代码才能运行，所以 recover 只在defer函数内部有用。

recover的一个应用是在服务器内部关闭一个失败的goroutine，而不杀死其他正在执行的goroutine。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

在这个例子中，如果do(work) panic，结果会被记录下来，goroutine会干净利落地退出，而不会打扰到其他程序。在 defer 闭包中不需要做任何其他事情，调用recover就可以完全处理这个条件。

因为除非直接从 defer 函数中调用 recover，否则 recover 总是返回nil，所以 defer 代码可以调用本身使用panic和recover的库例程而不会失败。举个例子，safeDo中的 defer 函数可能会在调用 recover 之前调用一个日志函数，而这个日志代码的运行不会受到 panic 状态的影响。

有了我们的 recovery 模式，do函数（以及它所调用的任何东西）可以通过调用 panic 来干净利落地摆脱任何糟糕的情况。我们可以用这个想法来简化复杂软件中的错误处理。让我们看看一个理想化版本的 regexp 包，它通过调用 panic 与本地错误类型来报告解析错误。下面是Error的定义、错误方法和Compile函数。

```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

如果doParse panic，recover 块将把返回值设置为nil- defer 函数可以修改命名的返回值。然后，它将在对err的赋值中，通过断言它具有本地类型Error来检查问题是否是解析错误。如果没有，类型断言将失败，导致运行时错误，继续堆栈展开，就像什么都没有中断一样。这个检查意味着，如果发生了意外的事情，比如索引出界，即使我们使用panic和recover来处理解析错误，代码也会失败。

有了错误处理，错误方法（因为它是一个绑定到类型的方法，所以它的名字和内置的错误类型相同是很好的，甚至是很自然的）就可以很容易地报告解析错误，而不用担心手动解开解析栈。

```go
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

虽然这个模式很有用，但它应该只在一个包内使用。Parse将其内部的 panic 调用转化为错误值；它不会将 panic 暴露给客户端。这是一个很好的规则。

顺便说一下，如果实际发生了错误，这个重新 panic 成语会改变panic值。然而，原始的和新的故障都会在崩溃报告中呈现，所以问题的根本原因仍然可见。因此，这种简单的重新panic方法通常已经足够了-- 毕竟是崩溃，但如果你想只显示原始值，你可以多写一点代码来过滤意外的问题，并用原始错误重新panic。这就留给读者去练习了。























