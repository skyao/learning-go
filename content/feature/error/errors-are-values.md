---
date: 2020-07-04T23:50:00+08:00
title: 错误是值(2015)
weight: 817
menu:
  main:
    parent: "feature-error"
description : "go中的错误是值"
---

> 备注：golang官方blog文章 [Errors are values](https://blog.golang.org/errors-are-values)

在Go程序员中，尤其是那些刚接触这门语言的程序员，经常讨论的一个问题就是如何处理错误。谈话往往变成了对这个序列的次数的哀叹：

```go
if err != nil {
    return err
}
```

显示出来。我们最近扫描了所有能找到的开源项目，发现这个片段每一两页只出现一次，比一些人认为的要少。不过，如果人们仍然认为必须键入：

```go
if err != nil
```

一直以来，一定有什么地方出了问题，而明显的目标是 go 本身。

这是不幸的，误导性的，而且很容易纠正。也许发生的情况是，刚接触 go 的程序员会问："如何处理错误？"。学会了这种模式，就止步于此。在其他语言中，人们可能会使用 try-catch 块或其他类似机制来处理错误。因此，程序员就会想，我在以前的语言中会使用 try-catch，在Go中我就直接输入 if err != nil。随着时间的推移，Go代码收集了很多这样的片段，结果感觉很笨拙。

不管这种解释是否合适，很明显，这些Go程序员忽略了一个关于错误的基本点：*Errors are values* （错误是值）。

值可以被编程，既然错误是值，那么错误也可以被编程。

当然，涉及错误值的常见语句是测试它是否为nil，但还有无数其他的事情可以用错误值来做，应用这些其他的一些事情可以让你的程序变得更好，消除了很多如果每个错误都用死板的 if 语句来检查所产生的模板。

下面是一个简单的例子，来自 bufio 包的 Scanner 类型。它的 Scan 方法执行了底层的I/O，这当然会导致错误。然而Scan方法根本没有暴露错误。相反，它返回一个布尔值，并在扫描结束时运行一个单独的方法，报告是否发生错误。客户端代码是这样的。

```go
scanner := bufio.NewScanner(input)
for scanner.Scan() {
    token := scanner.Text()
    // process token
}
if err := scanner.Err(); err != nil {
    // process the error
}
```

当然，有一个错误的nil检查，但它只出现和执行一次。扫描方法可以被定义为：

```go
func (s *Scanner) Scan() (token []byte, error)
```

然后示例用户代码可能是（取决于如何检索令牌）：

```go
scanner := bufio.NewScanner(input)
for {
    token, err := scanner.Scan()
    if err != nil {
        return err // or maybe break
    }
    // process token
}
```

这并没有什么不同，但有一个重要的区别。在这段代码中，客户端必须在每次迭代时检查错误，但在真正的 Scanner API 中，错误处理是从关键的 API 元素中抽象出来的，而关键的 API 元素就是迭代 tokens。因此，使用真正的API，客户端的代码感觉更自然：循环直到完成，然后再担心错误。错误处理不会掩盖控制流。

当然，在掩盖之下发生的事情是，一旦Scan遇到一个I/O错误，它就会记录下来并返回false。当客户端询问时，一个单独的方法Err会报告错误值。虽然这很微不足道，但它和把

```go
if err != nil
```

放的遍地都是，或者要求客户端在每个token之后检查错误。这就是带有错误值的编程。简单的编程，是的，但还是编程。

值得强调的是，无论设计如何，程序检查错误是至关重要的，无论它们是如何暴露的。这里讨论的不是如何避免检查错误，而是如何使用语言优雅地处理错误。

当我参加2014年秋季在东京举行的GoCon时，就出现了重复查错代码的话题。一位在Twitter上化名为@jxck_的热心地鼠对错误检查发出了熟悉的感叹。他有一些代码的示意是这样的。

```go
_, err = fd.Write(p0[a:b])
if err != nil {
    return err
}
_, err = fd.Write(p1[c:d])
if err != nil {
    return err
}
_, err = fd.Write(p2[e:f])
if err != nil {
    return err
}
// and so on
```

这是非常重复的。在真实的代码中，这段代码比较长，发生的事情比较多，所以不容易只用帮助函数重构，但在这种理想化的形式下，在错误变量上关联一个函数字面量会有帮助：

```go
var err error
write := func(buf []byte) {
    if err != nil {
        return
    }
    _, err = w.Write(buf)
}
write(p0[a:b])
write(p1[c:d])
write(p2[e:f])
// and so on
if err != nil {
    return err
}
```

这种模式很好用，但需要在每个做write的函数中都有一个闭包；单独的帮助函数使用起来比较笨拙，因为err变量需要在不同的调用中维护（试试）。

我们可以借鉴上面Scan方法的思想，使之更干净、更通用、更可重用。我在我们的讨论中提到了这个技术，但@jxck_并没有看到如何应用它。经过长时间的交流，由于语言障碍，我问是否可以借用他的笔记本，通过输入一些代码给他看。

我定义了一个叫 errWriter 的对象，类似这样。

```go
type errWriter struct {
    w   io.Writer
    err error
}
```

并给了它一个方法，write。它不需要有标准的 Write 签名，而且它的小写部分是为了突出区别。write方法会调用底层Writer的Write方法，并记录第一个错误，供以后参考。

```go
func (ew *errWriter) write(buf []byte) {
    if ew.err != nil {
        return
    }
    _, ew.err = ew.w.Write(buf)
}
```

一旦发生错误，写方法就会变成无操作，但错误值会被保存。

给定errWriter类型和它的写法，上面的代码可以重构。

```go
ew := &errWriter{w: fd}
ew.write(p0[a:b])
ew.write(p1[c:d])
ew.write(p2[e:f])
// and so on
if ew.err != nil {
    return ew.err
}
```

这样做更干净，甚至比使用闭包更干净，也使实际的写入顺序更容易在页面上看到。再也没有杂乱无章的东西了。用错误值（和接口）编程让代码变得更漂亮了。

很有可能在同一个包中的其他代码可以建立在这个想法上，甚至直接使用errWriter。

另外，一旦errWriter存在，它还可以做更多的事情来帮助我们，尤其是在不太人为的例子中。它可以累积字节数。它可以将写入的内容凝聚成一个单一的缓冲区，然后可以原子化地传输。还有更多。

事实上，这种模式经常出现在标准库中。archive/zip和net/http包都使用了它。更突出的是，bufio包的Writer实际上是errWriter思想的一个实现。虽然bufio.Writer.Write会返回一个错误，但那主要是为了尊重io.Writer接口。bufio.Writer的Write方法的行为就像我们上面的errWriter.write方法一样，Flush会报告错误，所以我们的例子可以这样写。

```go
b := bufio.NewWriter(fd)
b.Write(p0[a:b])
b.Write(p1[c:d])
b.Write(p2[e:f])
// and so on
if b.Flush() != nil {
    return b.Flush()
}
```

这种方法有一个重大的缺点，至少对某些应用来说是这样：无法知道错误发生前完成了多少处理。如果该信息很重要，就需要采用更细化的方法。不过，通常情况下，在最后进行全有或全无的检查就足够了。

我们只看了一种避免重复性错误处理代码的技术。请记住，使用 errWrite r或 bufio.Writer 并不是简化错误处理的唯一方法，而且这种方法并不适合所有情况。然而，关键的经验是，错误是值，可以利用Go编程语言的全部能力来处理它们。

使用该语言来简化你的错误处理。

但请记住。无论你做什么，总是要检查你的错误！

