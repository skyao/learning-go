---
date: 2020-07-04T23:50:00+08:00
title: 注释
weight: 411
menu:
  main:
    parent: "grammar-element"
---

### Go语言实战

> 摘录自 Go语言实战 3.5.3 Go 语言的文档一节

为了在 godoc 生成的文档里包含自己的代码文档，开发人员需要用下面的规则来写代码和注释。

在标识符之前，把文档作为注释加入到代码中，这个方式对包、函数、类型和全局变量都适用。注释可以以双斜线开头，也可以用斜线和星号风格：

```go
// Retrieve 连接到配置库,收集各种链接设置、用户名和密码。这个函数在成功时
// 返回 config 结构,否则返回一个错误。
func Retrieve() (config, error) {
	// ...省略
}
```

如果想给包写一段文字量比较大的文档,可以在工程里包含一个叫作 doc.go 的文件,使用同样的包名,并把包的介绍使用注释加在包名声明之前：

```go
/*
包 usb 提供了用于调用 USB 设备的类型和函数。想要与 USB 设备创建一个新链接,使用 NewConnection
...
*/
package usb
```

这段关于包的文档会显示在所有类型和函数文档之前。这个例子也展示了如何使用斜线和星 号做注释。

### Golang语言规范

https://golang.org/ref/spec#Comments

注释作为程序文档。有两种形式：

1. 行注释以字符序列 `//` 开始，止于行末。
2. 一般注释以字符序列 `/*` 开始，并以随后的第一个字符序列 `*/` 停止。

注释不能在符文或字符串字面值内部开始，也不能在注释内部开始。一个不包含换行符的一般注释就像一个空格。任何其他注释的作用就像一个换行符。

### Effective Go

https://golang.org/doc/effective_go.html#commentary

Go 提供了 C风格  `/* */` 块注释和 C++风格的 `//` 行注释。行注释是标准的；块注释主要是作为包注释出现的，但在表达式中或禁用大片代码时很有用。

godoc（程序和 web 服务器） 处理 Go 源文件，以提取有关包内容的文档。出现在顶层声明之前的注释，没有中间的换行符，与声明一起被提取出来，作为该项目的解释文本。这些注释的性质和风格决定了 godoc 产生的文档的质量。

每个包都应该有一个包注释，即在包子句之前的块注释。对于多文件的包，包注释只需要出现在一个文件中，任何一个文件都可以。包的注释应该介绍包，并提供与包整体相关的信息。它将首先出现在 godoc 页面上，并且应该构建后面的详细文档。

```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```

如果包的内容简单，包的注释可以简短一些。

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

注释不需要额外的格式化，比如星星的横幅。生成的输出可能甚至不会以固定宽度的字体呈现，所以不要依赖对齐的间距--godoc，像gofmt一样，会照顾到这一点。注释是未解释的纯文本，所以 HTML 和其他注释（如 `_this_`）会逐字重现，不应使用。godoc 做的一个调整是用固定宽度的字体显示缩进的文本，适合程序片段。fmt包的注释就很好地利用了这一点。

根据上下文的不同，godoc甚至可能不会重新格式化注释，所以确保它们直接看起来不错：使用正确的拼写、标点和句子结构，折叠长行，等等。

在一个包内，任何紧接在顶层声明之前的注释都会作为该声明的 doc 注释。程序中的每一个导出（大写）的名字都应该有一个 doc 注释。

doc注释最好是作为完整的句子来使用，这样就可以有各种各样的自动呈现方式。第一句话应该是一句话的总结，以被声明的名称开始。

```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

如果每个doc注释都以它所描述的物品名称开头，你可以使用go工具的doc子命令，通过grep运行输出。想象一下，你记不住 "Compile" 这个名字，但又在寻找正则表达式的解析函数，所以你运行了这个命令：

```bash
$ go doc -all regexp | grep -i parse
```

如果包中所有的 doc 注释都以 "This function..." 开头，grep 就不会帮你记住这个名字。但因为软件包中的每条文档注释都以名称开头，你会看到这样的内容，它能让你想起你要找的单词。

```bash
$ go doc -all regexp | grep -i parse
    Compile parses a regular expression and returns, if successful, a Regexp
    MustCompile is like Compile but panics if the expression cannot be parsed.
    parsed. It simplifies safe initialization of global variables holding
$
```

Go的声明语法允许对声明进行分组。一个文档注释可以介绍一组相关的常量或变量。由于整个声明都会被介绍，所以这样的注释往往可以敷衍了事。

```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

分组也可以表示项目之间的关系，比如一组变量是由一个mutex保护的。

```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

