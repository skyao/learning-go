---
date: 2020-07-04T23:50:00+08:00
title: 接口嵌入
weight: 865
menu:
  main:
    parent: "feature-interface"
description : "go语言的接口嵌入"
---

## Effective Go

https://golang.org/doc/effective_go.html#embedding

Go并没有提供典型的、类型驱动的子类概念，但它确实可以通过在结构体或接口中嵌入类型来 "借用"实现的一部分。

接口嵌入非常简单。我们之前已经提到了 io.Reader 和 io.Writer 接口，下面是它们的定义。

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

io包还导出了其他一些接口，这些接口指定了可以实现若干这样方法的对象。例如，有io.ReadWriter，一个包含Read和Write的接口。我们可以通过显式列出这两个方法来指定io.ReadWriter，但像这样把这两个接口嵌入形成新的接口，会更容易，也更有感召力。

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

如代码所示： ReadWriter可以做Reader做的事情，也可以做Writer做的事情；它是一个嵌入接口（必须是不相干的方法集）的联合体。只有接口可以嵌入到接口中。

同样的基本思想也适用于结构体，但其影响更为深远。bufio包有两个结构类型，bufio.Reader和bufio.Writer，当然每个结构都实现了包io中的类似接口。而且bufio还实现了一个缓冲的读/写器，它是通过使用嵌入的方式将一个读器和一个写器合并到一个结构中来实现的：它列出了结构中的类型，但没有给它们起字段名：

```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

嵌入的元素是指向结构体的指针，当然在使用之前必须初始化为指向有效的结构。ReadWriter结构可以写成：

```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

但这样一来，为了提升字段的方法以满足io接口，我们还需要提供转发方法，比如这样：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

通过直接嵌入结构，我们避免了这种方式。嵌入类型的方法直接附加，这意味着bufio.ReadWriter不仅拥有bufio.Reader和bufio.Writer的方法，还满足了所有三个接口：io.Reader、io.Writer和io.ReadWriter。

嵌入和子类有一个重要的区别。当我们嵌入一个类型时，该类型的方法会成为外部类型的方法，但当它们被调用时，方法的接收者是内部类型，而不是外部类型。在我们的例子中，当调用一个bufio.ReadWriter的Read方法时，它的效果和上面写出来的转发方法完全一样，接收者是ReadWriter的reader字段，而不是ReadWriter本身。

嵌入也可以是一种简单的方便。这个例子显示了一个嵌入字段与一个常规的、命名的字段并列。

```go
type Job struct {
    Command string
    *log.Logger
}
```

现在，Job类型有了*log.Logger的Print、Printf、Println等方法。当然，我们可以给Logger取一个字段名，但没有必要这么做。而现在，一旦初始化，我们就可以将日志记录到Job中。

```go
job.Println("starting now...")
```

Logger是Job结构的一个常规字段，所以我们可以在Job的构造函数中以通常的方式初始化它，像这样：

```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

或用组合字面量：

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

如果我们需要直接引用一个嵌入的字段，那么字段的类型名，忽略包的限定符，作为字段名，就像在我们的ReadWriter结构的Read方法中一样。在这里，如果我们需要访问一个Job变量job的*log.Logger，我们会写job.Logger，如果我们想完善Logger的方法，这将是非常有用的。

```go
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

嵌入类型引入了名称冲突的问题，但解决这些问题的规则很简单。首先，一个字段或方法X将任何其他项目X隐藏在类型的更深嵌套部分。如果log.Logger包含一个名为Command的字段或方法，那么Job的Command字段就会支配它。

其次，如果相同的名称出现在相同的嵌套层次，通常是一个错误；如果Job结构包含另一个名为Logger的字段或方法，那么嵌入log.Logger将是错误的。但是，如果重复的名称在程序中从未在类型定义之外提及，则是可以的。这个限定提供了一些保护，防止从外部对嵌入的类型进行修改；如果添加的字段与另一个子类型中的另一个字段发生冲突，如果两个字段都没有使用过，那么就没有问题。



