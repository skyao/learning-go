---
date: 2020-07-04T23:50:00+08:00
title: 标签作用域
weight: 461
menu:
  main:
    parent: "grammar-scope"
description : "go语言中的标签作用域"
---

### Label scope

https://golang.org/ref/spec#Label_scopes

标签由标签语句声明，并在 "break"、"continue "和 "goto "语句中使用。定义一个从未使用过的标签是非法的。与其他标识符不同的是，标签没有块范围，也不会与非标签的标识符冲突。

标签的作用域是它声明时所在的函数的主体，不包括任何嵌套函数的主体。