---
date: 2020-07-04T23:50:00+08:00
title: 表达式
weight: 500
description : "介绍Go语言的表达式"
---



## Expressions

https://golang.org/ref/spec#Expressions

表达式指定了通过对操作数应用运算符和函数来计算值。

### 操作数

操作数表示表达式中的基本值。操作数可以是字面量，一个代表常量、变量、函数的非空标识符（可能是限定的），或者带括号内的表达式。

空白的标识符只能作为操作数出现在赋值的左侧。

```
Operand     = Literal | OperandName | "(" Expression ")" .
Literal     = BasicLit | CompositeLit | FunctionLit .
BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
OperandName = identifier | QualifiedIdent .
```

### Qualified identifiers

限定的标识符是用包名前缀限定的标识符。包名和标识符都不能为空。

```
QualifiedIdent = PackageName "." identifier .
```

限定的标识符访问不同包中的标识符，这个标识符必须被导入。该标识符必须被导出，并在该包的包块中声明。

```go
math.Sin	// denotes the Sin function in package math
```