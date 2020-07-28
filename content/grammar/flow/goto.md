---
date: 2020-07-04T23:50:00+08:00
title: goto语句
weight: 609
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 goto 语句"
---

## golang语言规范

"goto" 语句将控制权转移到同一函数中带有相应标签的语句。

```
GotoStmt = "goto" Label .
```

```go
goto Error
```

执行 "goto"语句不能导致在goto之后有任何变量进入它还没有进行的范围。例如，这个例子：

```go
	goto L  // BAD
	v := 3
L:
```

是错误的，因为跳转到标签L时跳过了v的创建。

块外的 "goto"语句不能跳转到该块内的标签。例如，这个例子：

```go
if n%2 == 1 {
	goto L1
}
for n > 0 {
	f()
	n--
L1:
	f()
	n--
}
```

是错误的，因为标签L1在 "for"语句的块内，而goto却不在。