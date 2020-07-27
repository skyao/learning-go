---
date: 2020-07-04T23:50:00+08:00
title: 指针类型
weight: 477
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的指针类型"
---

## Pointer types

https://golang.org/ref/spec#Pointer_types

```
PointerType = "*" BaseType .
BaseType    = Type .
```

```go
*Point
*[4]int
```

### 参考资料

- [Dig101:Go 之聊聊 struct 的内存对齐](https://gocn.vip/topics/9759)
- [Go 之如何操作结构体的非导出字段](https://gocn.vip/topics/10553)
- https://gobyexample-cn.github.io/structs