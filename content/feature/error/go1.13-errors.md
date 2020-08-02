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

Go error 是值。程序根据这些值以几种方式做出决定。最常见的是将错误与nil进行比较，以确定操作是否失败。

```go
if err != nil {
    // something went wrong
}
```

有时我们会将错误与已知的 *sentinel* 值进行比较，看看是否发生了特定的错误。

```go
var ErrNotFound = errors.New("not found")

if err == ErrNotFound {
    // something wasn't found
}
```

错误值可以是满足语言定义的 error 接口的任何类型。程序可以使用类型断言或类型转换来将错误值视为更具体的类型。

```go
type NotFoundError struct {
    Name string
}

func (e *NotFoundError) Error() string { return e.Name + ": not found" }

if e, ok := err.(*NotFoundError); ok {
    // e.Name wasn't found
}
```

#### 添加信息

经常有函数在调用堆栈中传递错误，同时在其中添加信息，比如错误发生时的简要描述。一个简单的方法是构造新的错误，其中包括前一个错误的文本。

```go
if err != nil {
    return fmt.Errorf("decompress %v: %v", name, err)
}
```

使用 fmt.Errorf 创建一个新的错误，会丢弃原始错误中除文本以外的所有内容。正如我们在上面的QueryError中所看到的，我们有时可能希望定义一个新的错误类型，其中包含底层错误，保留它以便于代码检查。这里又是QueryError。

```go
type QueryError struct {
    Query string
    Err   error
}
```

程序可以在 `*QueryError` 值内部查看，根据底层 error 做出决定。你有时会看到这被称为 "解包 "错误。

```go
if e, ok := err.(*QueryError); ok && e.Err == ErrPermission {
    // query failed because of a permission problem
}
```

标准库中的os.PathError类型是另一个例子，它包含了一个错误。

### Errors in Go 1.13

#### Unwrap 方法

Go 1.13 为 errors 和 fmt 标准库包引入了新特性，以简化对包含其他错误的错误的处理。其中最重要的是约定，而不是变化：包含另一个error的 error 可以实现一个返回底层 error 的 Unwrap 方法。如果 e1.Unwrap() 返回 e2，那么我们就说 e1 封装(wrap)了e2，可以解开 (unwrap) e1 得到 e2。

按照这个约定，我们可以给上面的 QueryError 类型一个 Unwrap 方法，返回其包含的错误。

```go
func (e *QueryError) Unwrap() error { return e.Err }
```

#### 使用 Is 和 As 检查错误

Go 1.13 errors 包包含了两个新的错误检查函数：Is 和 As。

errors.Is 函数将错误与值进行比较：

```go
// Similar to:
//   if err == ErrNotFound { … }
if errors.Is(err, ErrNotFound) {
    // something wasn't found
}
```

As 函数测试 error 是否为特定类型：

```go
// Similar to:
//   if e, ok := err.(*QueryError); ok { … }
var e *QueryError
if errors.As(err, &e) {
    // err is a *QueryError, and e is set to the error's value
}
```

在最简单的情况下，errors.Is 函数的行为就像与 sentinel error 的比较，而 errors.As 函数的行为就像类型断言。然而，当对封装的错误进行操作时，这些函数会考虑链中的所有error。让我们再看看上面的例子，即解开一个 QueryError 来检查底层错误。

```go
if e, ok := err.(*QueryError); ok && e.Err == ErrPermission {
    // query failed because of a permission problem
}
```

使用 errors.Is 函数，我们可以把它写成：

```go
if errors.Is(err, ErrPermission) {
    // err, or some error that it wraps, is a permission problem
}
```

errors 包还包括一个新的 Unwrap 函数，它返回调用 error 的 Unwrap 方法的结果，如果错误没有Unwrap方法，则返回 nil。但通常最好使用 errors.Is 或 errors.As，因为这些函数会在一次调用中检查整个链。

#### 使用 %w 封装错误

如前所述，通常使用 fmt.Errorf 函数为错误添加附加信息。

```go
if err != nil {
    return fmt.Errorf("decompress %v: %v", name, err)
}
```

在Go 1.13中，fmt.Errorf 函数支持一个新的 %w 动词。当这个动词存在时，fmt.Errorf 返回的错误将有一个 Unwrap 方法返回 %w 的参数，它必须是一个error。在所有其他方面，%w 与 %v 相同：

```go
if err != nil {
    // Return an error which unwraps to err.
    return fmt.Errorf("decompress %v: %w", name, err)
}
```

用 %w 包装 error ，使得它可以被 errors.Is 和 erros.As 使用。

```go
err := fmt.Errorf("access denied: %w", ErrPermission)
...
if errors.Is(err, ErrPermission) ...
```

#### 是否封装

当向 error 添加额外的上下文时，无论是使用 fmt.Errorf 还是通过实现自定义类型，您都需要决定新的 error 是否应该封装原始 error。这个问题没有唯一的答案，它取决于创建新 error 的上下文。封装一个error 以将其暴露给调用者。如果这样做会暴露实现细节，则不要包装error。

举个例子，想象一个从 io.Reader.Reader 中读取复杂数据结构的 Parsse 函数。如果发生错误，我们希望报告发生错误的行号和列号。如果错误是在从 io.Reader 读取时发生的，我们希望对该错误进行包装，以便检查潜在的问题。由于调用者向函数提供了 io.Reader，所以暴露它所产生的错误是有意义的。

相反，一个对数据库进行多次调用的函数可能不应该返回一个对其中一次调用结果进行解包的错误。如果函数使用的数据库是一个实现细节，那么暴露这些错误就违反了抽象性。例如，如果你的包pkg的LookupUser函数使用了Go的数据库/sql包，那么它可能会遇到一个sql.ErrNoRows错误。如果你用fmt.Errorf("accessing DB: %v", err)来返回这个错误，那么调用者就不能在里面查找sql.ErrNoRows。但是如果函数返回的是fmt.Errorf("accessing DB: %w", err)，那么调用者就可以合理地写道

```go
err := pkg.LookupUser(...)
if errors.Is(err, sql.ErrNoRows) …
```

这时，如果你不想破坏你的客户端，即使你切换到不同的数据库包，函数必须总是返回 sql.ErrNoRows 。换句话说，包装一个错误使该错误成为你的API的一部分。如果你不想承诺在未来支持该错误作为你的API的一部分，你就不应该包装该错误。

重要的是要记住，无论你是否封装，错误文本都是一样的。试图理解该错误的人无论用哪种方式都会得到相同的信息；选择封装是为了给程序提供额外的信息，以便他们能够做出更明智的决定，还是为了保留抽象层而不提供该信息。

#### 使用Is和As方法自定义错误测试

errors.Is 函数检查链中每个错误是否与目标值匹配。默认情况下，如果两者相等，则错误与目标值匹配。此外，链中的错误可以通过实现 Is 方法声明它与目标值匹配。

作为一个例子，考虑这个错误的灵感来自 Upspin 错误包，它将错误与模板进行比较，只考虑模板中非零的字段。

```go
type Error struct {
    Path string
    User string
}

func (e *Error) Is(target error) bool {
    t, ok := target.(*Error)
    if !ok {
        return false
    }
    return (e.Path == t.Path || t.Path == "") &&
           (e.User == t.User || t.User == "")
}

if errors.Is(err, &Error{User: "someuser"}) {
    // err's User field is "someuser".
}
```

errors.As 函数同样在存在的情况下咨询 As 方法。

#### Errors 和包API

返回错误的包（大多数都是这样）应该描述这些错误的属性，程序员可以依赖这些属性。一个设计良好的包也会避免返回具有不应该依赖的属性的错误。

最简单的规范是说，操作要么成功，要么失败，分别返回一个 nil 或 non-nil 的错误值。在很多情况下，不需要进一步的信息。

如果我们希望函数返回一个可识别的错误条件，比如 "item not found"，我们可能会返回一个包裹着 sentinel 的 error。

```go
var ErrNotFound = errors.New("not found")

// FetchItem returns the named item.
//
// If no item with the name exists, FetchItem returns an error
// wrapping ErrNotFound.
func FetchItem(name string) (*Item, error) {
    if itemNotFound(name) {
        return nil, fmt.Errorf("%q: %w", name, ErrNotFound)
    }
    // ...
}
```

还有其他现有的模式可以提供可以被调用者进行语义检查的错误，例如直接返回一个哨兵值、一个特定的类型或一个可以用谓词函数检查的值。

在所有情况下，都应该注意不要向用户暴露内部细节。正如我们在上面的 "是否封装" 中提到的，当你从另一个包中返回一个错误时，你应该将错误转换为不暴露底层错误的形式，除非你愿意承诺在将来返回那个特定的错误。

```go
f, err := os.Open(filename)
if err != nil {
    // The *os.PathError returned by os.Open is an internal detail.
    // To avoid exposing it to the caller, repackage it as a new
    // error with the same text. We use the %v formatting verb, since
    // %w would permit the caller to unwrap the original *os.PathError.
    return fmt.Errorf("%v", err)
}
```

如果函数被定义为返回一个包裹着某个哨兵或类型的 error ，不要直接返回底层错误。

```go
var ErrPermission = errors.New("permission denied")

// DoSomething returns an error wrapping ErrPermission if the user
// does not have permission to do something.
func DoSomething() error {
    if !userHasPermission() {
        // If we return ErrPermission directly, callers might come
        // to depend on the exact error value, writing code like this:
        //
        //     if err := pkg.DoSomething(); err == pkg.ErrPermission { … }
        //
        // This will cause problems if we want to add additional
        // context to the error in the future. To avoid this, we
        // return an error wrapping the sentinel so that users must
        // always unwrap it:
        //
        //     if err := pkg.DoSomething(); errors.Is(err, pkg.ErrPermission) { ... }
        return fmt.Errorf("%w", ErrPermission)
    }
    // ...
}
```

### 总结

虽然我们所讨论的变化仅仅是三个函数和一个格式化动词，但我们希望它们将大大改善Go程序中的错误处理方式。我们希望通过包装来提供额外的上下文会变得很普遍，帮助程序做出更好的决策，帮助程序员更快地找到错误。

正如Russ Cox在GopherCon 2019的主题演讲中所说，在通往Go 2的道路上，我们进行实验、简化和出货。现在我们已经出货了这些变化，我们期待着接下来的实验。









































































