---
date: 2020-07-04T23:50:00+08:00
title: 常量声明
weight: 452
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的Getter"
---

### Constant declaration

https://golang.org/ref/spec#Constant_declarations

常量声明将标识符列表（常量的名称）与常量表达式列表的值绑定。标识符的数量必须等于表达式的数量，左边的第n个标识符与右边的第n个表达式的值绑定。

```go
ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .

IdentifierList = identifier { "," identifier } .
ExpressionList = Expression { "," Expression } .
```

如果存在类型，所有常量都采用指定的类型，并且表达式对于该类型必须是 assignable /可分配的。如果类型被省略，则常量取对应表达式各自的类型。如果表达式的值是无类型的常量，则声明的常量保持无类型，而常量标识符表示常量值。例如，如果表达式是浮点字面量，常量标识符表示浮点常量，即使字面量的小数部分为零。

```go
const Pi float64 = 3.14159265358979323846
const zero = 0.0         // untyped floating-point constant
const (
	size int64 = 1024
	eof        = -1  // untyped integer constant
)
const a, b, c = 3, 4, "foo"  // a = 3, b = 4, c = "foo", untyped integer and string constants
const u, v float32 = 0, 3    // u = 0.0, v = 3.0
```

在带括号的const声明列表中，除了第一个ConstSpec之外，表达式列表可以省略。这样的空列表相当于前面第一个非空的表达式列表及其类型（如果有的话）的文本替换。因此，省略表达式列表相当于重复前面的列表。标识符的数量必须等于前一个列表中表达式的数量。与iota常量生成器一起，这种机制允许轻量级的顺序值声明。

```go
const (
	Sunday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Partyday
	numberOfDays  // this constant is not exported
)
```