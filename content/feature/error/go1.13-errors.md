---
date: 2020-07-04T23:50:00+08:00
title: go1.13中的错误处理(2019)
weight: 819
menu:
  main:
    parent: "feature-error"
description : "go1.13中的错误处理"
---

> 备注：golang官方blog文章 [Working with Errors in Go 1.13](https://blog.golang.org/go1.13-errors)

### 介绍

在过去的十年里，Go 将错误处理为值（[errors as values](https://blog.golang.org/errors-are-values)）的做法对我们很有帮助。虽然标准库对错误的支持很少--只有 errors.New 和 fmt.Errorf 函数，它们产生的错误只包含一条消息--但内置的 error 接口允许 Go 程序员添加他们想要的任何信息。它所需要的只是一个实现 Error 方法的类型。

```go
type QueryError struct {
    Query string
    Err   error
}

func (e *QueryError) Error() string { return e.Query + ": " + e.Err.Error() }
```

像这样的错误类型无处不在，它们所存储的信息也千差万别，从时间戳到文件名到服务器地址。通常，这些信息包括另一个低级错误，以提供额外的上下文。

一个错误包含另一个错误的模式在 Go 代码中是如此普遍，以至于经过广泛的讨论，Go 1.13 增加了对它的明确支持。这篇文章描述了标准库中提供这种支持的新增内容：错误包中的三个新函数，以及 fmt.Errorf 的新格式动词。

在详细描述这些变化之前，让我们先回顾一下在以前的语言版本中是如何检查和构造错误的。

### go 1.13之前的错误

#### 检查错误





