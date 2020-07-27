---
date: 2020-07-04T23:50:00+08:00
title: 声明
weight: 450
description : "介绍Go语言的声明"
---

### 声明和作用域

https://golang.org/ref/spec#Declarations_and_scope

声明将一个非空的标识符绑定到常量、类型、变量、函数、标签或包上。程序中的每个标识符都必须声明。任何标识符都不能在同一个块中声明两次，标识符也不能在文件和包块中同时声明。

空白标识符可以像其他标识符一样在声明中使用，但它不引入绑定，因此不声明。在包块中，标识符init只能用于init函数的声明，和空白标识符一样，它不会引入新的绑定。

```go
Declaration   = ConstDecl | TypeDecl | VarDecl .
TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .
```

