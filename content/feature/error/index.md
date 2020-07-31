---
date: 2020-07-04T23:50:00+08:00
title: 错误处理
weight: 900
description : "介绍Golang的错误处理"
---

## go语言实战

符合 Go 语言习惯的做法是使用一个独立、明确的返回值来传递错误信息。 

这与 Java、Ruby 使用的异常（exception） 以及在 C 语言中有时用到的重载 (overloaded) 的单返回/错误值有着明显的不同。 

Go 语言的处理方式能清楚的知道哪个函数返回了错误，并使用跟其他（无异常处理的）语言类似的方式来处理错误。

```go
// 按照惯例，错误通常是最后一个返回值并且是 error 类型，它是一个内建的接口。
func f1(arg int) (int, error) { // 
    if arg == 42 {
        // errors.New 使用给定的错误信息构造一个基本的 error 值。
        return -1, errors.New("can't work with 42")
    }
    // 返回错误值为 nil 代表没有错误。
    return arg + 3, nil
}
```

你还可以通过实现 Error() 方法来自定义 error 类型。 这里使用自定义错误类型来表示上面例子中的参数错误:

```go
// 使用自定义错误类型来表示上面例子中的参数错误
type argError struct {
    arg  int
    prob string
}
// 通过实现 Error() 方法来自定义 error 类型
func (e *argError) Error() string {
    return fmt.Sprintf("%d - %s", e.arg, e.prob)
}

func f2(arg int) (int, error) {
    if arg == 42 {
         // 使用 &argError 语法来建立一个新的结构体
         // 并提供了 arg 和 prob 两个字段的值
         return -1, &argError{arg, "can't work with it"}
    }
    return arg + 3, nil
}
```

## Effective Go

https://golang.org/doc/effective_go.html#errors

类库例程必须经常向调用者返回某种错误指示。如前所述，Go的多值返回使得在返回正常返回值的同时很容易返回一个详细的错误说明。利用这个特性来提供详细的错误信息是一种很好的风格。例如，正如我们将看到的，os.Open 并不只是在失败时返回一个 nil 指针，它还返回一个错误值来描述出了什么问题。

按照惯例，错误的类型为error，是一个简单的内置接口。

```go
type error interface {
    Error() string
}
```

类库编写者可以自由地用更丰富的模型来实现这个接口，使其不仅可以看到错误，还可以提供一些上下文。如前所述，除了通常的*os.File返回值之外，os.Open还返回一个错误值。如果文件被成功打开，错误值将为nil，但当出现问题时，它将持有一个os.PathError。

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

PathError的Error会生成这样一个字符串：

```go
open /etc/passwx: no such file or directory
```

这样的错误包括有问题的文件名、操作和它所触发的操作系统错误，即使打印出来的时候离引起它的调用很远也是有用的；它比单纯的 "没有这样的文件或目录 "信息量大得多。

在可行的情况下，错误字符串应该识别它们的来源，例如通过有一个前缀来命名产生错误的操作或包。例如，在包image中，由于未知格式导致的解码错误的字符串表示是 "image: unknown format"。

关心精确错误细节的调用者可以使用类型开关（type switch）或类型断言（type assertion）来查找特定错误并提取细节。对于PathErrors来说，这可能包括检查内部Err字段是否存在可恢复的故障。

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

这里的第二个 if 语句是另一种类型的断言。如果它失败了，ok将为false，e将为nil. 如果它成功了，ok将为true，这意味着错误类型为*os.PathError，那么e也是如此，我们可以检查更多信息。



