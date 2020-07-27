---
date: 2020-07-04T23:50:00+08:00
title: Token
weight: 415
menu:
  main:
    parent: "grammar-element"
description : "go语言中的Token"
---

> 备注：摘录自 golang语言规范 https://golang.org/ref/spec#Tokens

### Token

Token构成了go语言的词汇。有四类Token：*identifiers*（标识符）、*keywords*（关键字）、*operators and punctuation*（运算符和标点符号）、以及*literals*（字面量）。

由空格(U+0020)、水平制表符(U+0009)、回车符(U+000D)和换行符(U+000A)形成的空白（whitespace）会被忽略，除非它是用来分隔Token，避免多个Token合并成一个Token。

此外，换行或文件末尾可能会触发分号的插入。

在将输入内容分解成Token的同时，下一个Token是形成有效Token的最长字符序列。

### 分号

在一些产品中正式语法使用分号"; "作为终结符。Go程序可以使用以下两条规则来省略大部分的分号：

1. 当输入的内容被分解成Token时，分号会自动插入到Token流中，紧接在一行的最后一个Token之后，如果该Token是... ...

	* identifier/标识符
	* 整型，浮点，虚数，rune（符文）或者 字符串字面值
	* break、continue、fallthrough、return等关键字之一
	* 运算符和标点符号 `++, --, ), ], 或 }` 中的一种

2. 为了让复杂的语句只占一行，可以在结尾") "或"}"前省略分号。



