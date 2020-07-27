---
date: 2020-07-04T23:50:00+08:00
title: string类型
weight: 473
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的string类型"
---

### String types

https://golang.org/ref/spec#String_types

字符串类型表示字符串值的集合。一个字符串值是一个（可能是空的）字节序列。字节数称为字符串的length/长度，永远不会是负数。字符串是不可改变的：一旦创建，就不可能改变字符串的内容。预先声明的字符串类型是字符串，它是已定义类型。

字符串s的长度可以通过内置函数len来发现。如果字符串是常量，那么长度就是编译时常量。字符串的字节可以通过整数索引0到len(s)-1来访问。取这种元素的地址是非法的；如果 s[i] 是字符串的第 i'th 个字节，&s[i] 是无效的。



### 参考资料

- [Dig101:Go 之 string 那些事](https://gocn.vip/topics/9593)





