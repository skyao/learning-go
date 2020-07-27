---
date: 2020-07-04T23:50:00+08:00
title: 类型声明
weight: 454
menu:
  main:
    parent: "grammar-declaration"
description : "go语言中的类型声明"
---

### Type declaration

https://golang.org/ref/spec#Type_declarations

类型声明将标识符，即类型名称，绑定到类型上。类型声明有两种形式：别名声明和类型定义。

```go
TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec = AliasDecl | TypeDef .
```

#### Alias declarations

别名声明将标识符绑定到给定的类型上：

```go
AliasDecl = identifier "=" Type .
```

在标识符的范围内，它作为类型的别称：

```go
type (
	nodeList = []*Node  // nodeList and []*Node are identical types
	Polar    = polar    // Polar and polar denote identical types
)
```

#### Type definitions

类型定义创建了一个新的、独特的类型，它与给定的类型具有相同的底层类型和操作，并为它绑定了一个标识符。

```
TypeDef = identifier Type .
```

新类型被称为 *defined type* / 定义类型。它不同于任何其他类型，包括它创建时来源的类型。

```go
type (
	Point struct{ x, y float64 }  // Point and struct{ x, y float64 } are different types
	polar Point                   // polar and Point denote different types
)

type TreeNode struct {
	left, right *TreeNode
	value *Comparable
}

type Block interface {
	BlockSize() int
	Encrypt(src, dst []byte)
	Decrypt(src, dst []byte)
}
```

defined type / 定义类型可以有与之相关的方法。它不继承任何与给定类型绑定的方法，但接口类型的方法集或复合类型的元素保持不变。

```go
// A Mutex is a data type with two methods, Lock and Unlock.
type Mutex struct         { /* Mutex fields */ }
func (m *Mutex) Lock()    { /* Lock implementation */ }
func (m *Mutex) Unlock()  { /* Unlock implementation */ }

// NewMutex has the same composition as Mutex but its method set is empty.
type NewMutex Mutex

// The method set of PtrMutex's underlying type *Mutex remains unchanged,
// but the method set of PtrMutex is empty.
type PtrMutex *Mutex

// The method set of *PrintableMutex contains the methods
// Lock and Unlock bound to its embedded field Mutex.
type PrintableMutex struct {
	Mutex
}

// MyBlock is an interface type that has the same method set as Block.
type MyBlock Block
```

类型定义可以用来定义不同的布尔、数值或字符串类型，并与它们相关联的方法：

```go
type TimeZone int

const (
	EST TimeZone = -(5 + iota)
	CST
	MST
	PST
)

func (tz TimeZone) String() string {
	return fmt.Sprintf("GMT%+dh", tz)
}
```