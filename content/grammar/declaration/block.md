---
date: 2020-07-04T23:50:00+08:00
title: Block
weight: 451
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的Getter"
---

> 备注：摘录自 Golang语言规范 https://golang.org/ref/spec#Blocks

### Blocks

Block/块是在匹配的大括号内的声明和语句序列（可能为空）：

```go
Block = "{" StatementList "}" .
StatementList = { Statement ";" } .
```

除了源码中的显性块，还有隐性块：

- *universe block* （宇宙块）包含了所有的go源代码文本。
- 每个包都有一个 package block （包块），包含该包的所有go源代码文本。
- 每个文件都有一个 file block （文件块），包含该文件中的所有go源代码文本。
- 每个 "if"、"for "和 "switch "语句都被认为是在自己的隐含块中。
- 在 "switch "或 "select "语句中的每个子句都作为一个隐式块。

块可以嵌套，会影响scope/范围。