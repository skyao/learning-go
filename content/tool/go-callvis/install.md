---
date: 2018-12-04T23:50:00+08:00
title: go-callvis安装
weight: 4041
menu:
  main:
    parent: "tool-go-callvis"
description : "go-callvis安装"
---

### 准备

参考 https://github.com/ofabry/go-callvis 的要求，需要先准备：

- go 1.12+
- [Graphviz](http://www.graphviz.org/download/)

因此需要先安装 graphviz，否则运行时会报错找不到"dot"。

- mac下：直接运行 `brew install graphviz`

### 安装

运行命令，注意GOPATH 要正确设置好：

```bash
go get -u github.com/ofabry/go-callvis
```

### 验证

参考官方例子：https://github.com/ofabry/go-callvis/tree/master/examples





