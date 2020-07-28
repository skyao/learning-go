---
date: 2020-07-04T23:50:00+08:00
title: break语句
weight: 607
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 break 语句"
---

## golang语言规范

https://golang.org/ref/spec#Break_statements

"break "语句终止了同一函数中最内层的 "for"、"switch "或 "select "语句的执行。

```
BreakStmt = "break" [ Label ] .
```

如果有标签，则标签必须包围住 "for"、"switch "或 "select "语句，而且标签是执行终止。









