---
date: 2020-07-04T23:50:00+08:00
title: 错误处理与go(2011)
weight: 816
menu:
  main:
    parent: "feature-error"
description : "错误处理与go"
---

> 备注：golang官方blog文章 [Error handling and Go](https://blog.golang.org/error-handling-and-go)

 ### 介绍

只要写过任何Go代码，就可能遇到过内置的 error 类型。Go代码使用错误值来表示异常状态。例如，os.Open函数在打开文件失败时，会返回一个非零的错误值：

```go
func Open(name string) (file *File, err error)
```

下面的代码使用 os.Open 来打开一个文件，如果发生错误，则调用 log.Fatal 来打印错误信息并停止。

```go
f, err := os.Open("filename.ext")
if err != nil {
    log.Fatal(err)
}
// do something with the open *File f
```

只知道错误类型这一点，你就可以在Go中完成很多工作，但在这篇文章中，我们将仔细研究错误，并讨论一些在Go中处理错误的好做法。

### error 类型

error 类型是一种接口类型。error 变量代表任何可以描述为字符串的值。下面是接口的声明。

```go
type error interface {
    Error() string
}
```

与所有内置类型一样，error 类型在宇宙块中预先声明。

最常用的 error 实现是 error package 的未导出的 errorString 类型。

```go
// errorString is a trivial implementation of error.
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}
```

可以使用 errors.New 函数来构造这些值。它接收一个字符串，并将其转换为 errors.errorString，然后作为一个错误值返回。

```go
// New returns an error that formats as the given text.
func New(text string) error {
    return &errorString{text}
}
```

下面是如何使用 errors.New：

```go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // implementation
}
```

传递负参数给Sqrt的调用者会收到一个非零的错误值（其具体表示是一个error.errorString值）。调用者可以通过调用错误的Error方法来访问错误字符串("math: square root of...")，或者直接打印它。

```go
f, err := Sqrt(-1)
if err != nil {
    fmt.Println(err)
}
```

fmt包通过调用 Error() 字符串方法来格式化错误值。

错误实现的责任是总结上下文。os.Open 返回的错误格式为 "open /etc/passwd: permission denied"，而不仅仅是 "permission denied."。我们的Sqrt返回的错误缺少了无效参数的信息。

要添加这些信息，一个有用的函数是fmt包的Errf。它根据Printf的规则格式化一个字符串，并将其作为一个由error.New创建的错误返回。

```go
if f < 0 {
    return 0, fmt.Errorf("math: square root of negative number %g", f)
}
```

在很多情况下，fmt.Errorf已经足够好了，但是由于error是一个接口，你可以使用任意的数据结构作为错误值，以允许调用者检查错误的细节。

例如，我们假设的调用者可能想恢复传递给Sqrt的无效参数。我们可以通过定义一个新的错误实现而不是使用 errors.errorString 来实现。

```go
type NegativeSqrtError float64

func (f NegativeSqrtError) Error() string {
    return fmt.Sprintf("math: square root of negative number %g", float64(f))
}
```

然后，复杂的调用者可以使用类型断言来检查NegativeSqrtError，并对其进行特殊处理，而只是将错误传递给fmt.Println或log.Fatal的调用者则不会看到行为的改变。

另一个例子是，json包指定了一个SyntaxError类型，当json.Decode函数在解析JSON blob时遇到语法错误时，它会返回这个类型。

```go
type SyntaxError struct {
    msg    string // description of error
    Offset int64  // error occurred after reading Offset bytes
}

func (e *SyntaxError) Error() string { return e.msg }
```

Offset字段甚至没有显示在错误的默认格式中，但调用者可以使用它来为他们的错误信息添加文件和行信息。

```go
if err := dec.Decode(&val); err != nil {
    if serr, ok := err.(*json.SyntaxError); ok {
        line, col := findLine(f, serr.Offset)
        return fmt.Errorf("%s:%d:%d: %v", f.Name(), line, col, err)
    }
    return err
}
```

这是Camlistore项目中一些实际代码的略微简化版本）。

错误接口只需要一个Error方法；特定的错误实现可能有额外的方法。例如，net包按照通常的惯例返回类型为error的错误，但一些错误实现有net.Error接口定义的附加方法。

```go
package net

type Error interface {
    error
    Timeout() bool   // Is the error a timeout?
    Temporary() bool // Is the error temporary?
}
```

客户端代码可以通过类型断言来测试net.Error，然后区分暂时性的网络错误和永久性的错误。例如，网络爬虫可能会在遇到暂时性错误时休眠并重试，否则就会放弃。

```go
if nerr, ok := err.(net.Error); ok && nerr.Temporary() {
    time.Sleep(1e9)
    continue
}
if err != nil {
    log.Fatal(err)
}
```

### 简化重复性错误处理

在Go中，错误处理很重要。该语言的设计和约定鼓励您在错误发生时明确地检查错误（与其他语言中的抛出异常和有时捕获错误的约定不同）。在某些情况下，这使得Go代码变得啰嗦，但幸运的是，您可以使用一些技术来减少重复的错误处理。

考虑一个带有 HTTP 处理程序的 App Engine 应用程序，该处理程序从数据存储中检索一条记录，并使用模板对其进行格式化。

```go
func init() {
    http.HandleFunc("/view", viewRecord)
}

func viewRecord(w http.ResponseWriter, r *http.Request) {
    c := appengine.NewContext(r)
    key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
    record := new(Record)
    if err := datastore.Get(c, key, record); err != nil {
        http.Error(w, err.Error(), 500)
        return
    }
    if err := viewTemplate.Execute(w, record); err != nil {
        http.Error(w, err.Error(), 500)
    }
}
```

这个函数处理由datastore.Get函数和viewTemplate的Execute方法返回的错误。在这两种情况下，它都会向用户呈现一个简单的错误信息，并给出HTTP状态码500（"内部服务器错误"）。这看起来是一个可管理的代码量，但增加一些HTTP处理程序，你很快就会得到许多相同错误处理代码的副本。

为了减少重复，我们可以定义自己的HTTP appHandler类型，其中包括一个错误返回值：

```go
type appHandler func(http.ResponseWriter, *http.Request) error
```

然后，我们可以改变我们的viewRecord函数来返回错误。

```go
func viewRecord(w http.ResponseWriter, r *http.Request) error {
    c := appengine.NewContext(r)
    key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
    record := new(Record)
    if err := datastore.Get(c, key, record); err != nil {
        return err
    }
    return viewTemplate.Execute(w, record)
}
```

这比原来的版本更简单，但http包并不理解返回错误的函数。为了解决这个问题，我们可以在appHandler上实现http.Handler接口的ServeHTTP方法。

```go
func (fn appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if err := fn(w, r); err != nil {
        http.Error(w, err.Error(), 500)
    }
}
```

ServeHTTP方法调用appHandler函数，并向用户显示返回的错误（如果有的话）。请注意，该方法的接收器 fn 是一个函数，（Go 可以做到这一点！）该方法通过调用表达式 fn(w) 中的接收者来调用函数。(Go 可以做到这一点！) 方法通过调用表达式 fn(w, r) 中的接收者来调用函数。

现在，当我们在http包中注册viewRecord时，我们使用Handle函数（而不是HandleFunc），因为appHandler是一个http.Handler（而不是http.HandlerFunc）。

```go
func init() {
    http.Handle("/view", appHandler(viewRecord))
}
```

有了这个基本的错误处理基础架构，我们可以让它变得更加友好。与其仅仅显示错误字符串，不如给用户一个简单的错误信息，并附上适当的HTTP状态代码，同时将完整的错误记录到App Engine开发者控制台，以便进行调试。

为此，我们创建一个appError结构，包含一个错误和一些其他字段。

```go
type appError struct {
    Error   error
    Message string
    Code    int
}
```

接下来我们修改appHandler类型来返回*appError值。

```go
type appHandler func(http.ResponseWriter, *http.Request) *appError
```

(通常情况下，传回 error 的具体类型而不是 error 是错误的，原因在Go FAQ中讨论过，但在这里是正确的，因为ServeHTTP是唯一能看到该值并使用其内容的地方。)

并让appHandler的ServeHTTP方法将appError的Message以正确的HTTP状态码显示给用户，并将完整的Error记录到开发者控制台。

```go
func (fn appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if e := fn(w, r); e != nil { // e is *appError, not os.Error.
        c := appengine.NewContext(r)
        c.Errorf("%v", e.Error)
        http.Error(w, e.Message, e.Code)
    }
}
```

最后，我们将viewRecord更新为新的函数签名，并让它在遇到错误时返回更多的上下文。

```go
func viewRecord(w http.ResponseWriter, r *http.Request) *appError {
    c := appengine.NewContext(r)
    key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
    record := new(Record)
    if err := datastore.Get(c, key, record); err != nil {
        return &appError{err, "Record not found", 404}
    }
    if err := viewTemplate.Execute(w, record); err != nil {
        return &appError{err, "Can't display record", 500}
    }
    return nil
}
```

这个版本的viewRecord和原来的长度是一样的，但现在每一行都有特定的意义，我们提供的是更友好的用户体验。

这并没有结束，我们还可以进一步改进我们应用程序中的错误处理。一些想法。

- 给错误处理程序一个漂亮的HTML模板。

- 当用户是管理员时，通过将堆栈跟踪写入HTTP响应，使调试变得更容易。

- 为appError写一个构造函数，存储堆栈跟踪以方便调试。

- 从appHandler内部的恐慌中恢复，将错误记录到控制台中，称为 "Critical"，同时告诉用户 "发生了一个严重的错误"。这是一个很好的触动，避免了让用户暴露在编程错误引起的不可捉摸的错误信息中。更多细节请参见Defer、Panic和Recover文章。

### 结束语

正确的错误处理是优秀软件的基本要求。通过运用本篇文章中描述的技术，你应该能够写出更可靠、更简洁的Go代码。