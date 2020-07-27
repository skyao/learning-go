---
date: 2020-07-04T23:50:00+08:00
title: struct类型
weight: 476
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的结构体类型"
---

## Struct

> 摘录自 go语言实战

结构体是字段的集合。结构体定义的语法：

```go
type Vertex struct {
	X int
	Y int
}
```

访问范围通过结构体和字段名的首字母大小写来体现：大写为public，小写为private。

### 构建结构体

可以通过值列表给结构体的各个字段赋值，新分配一个结构体，顺序和结构体定义的字段顺序一致。也可以通过使用 `Name:` 语法为单个字段赋值，未明确赋值的字段则取缺省值。

```go
var (
	v1 = Vertex{1, 2}  // 类型为 Vertex
	v2 = Vertex{X: 1}  // Y:0 被省略
	v3 = Vertex{}      // X:0 和 Y:0
	p  = &Vertex{1, 2} // 类型为 *Vertex
)
```

特殊的前缀 `&` 返回一个指向结构体的指针。

### 访问字段

通过点号访问结构体的字段：

```go
v := Vertex{1, 2}
v.X = 4
fmt.Println(v.X)
```

也可以通过指针访问：

```go
v := Vertex{1, 2}
p := &v
p.X = 1e9
fmt.Println(v)
```

## Struct types

https://golang.org/ref/spec#Struct_types

结构体是命名元素（称为字段）的序列，每个字段都有name/名称和type/类型。字段名可以显式（IdentifierList）或隐式（EmbeddedField）指定。在结构体中，非空白的字段名必须是唯一的。

```
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName .
Tag           = string_lit .
```

```go
// An empty struct.
struct {}

// A struct with 6 fields.
struct {
	x, y int
	u float32
	_ float32  // padding
	A *[]int
	F func()
}
```

一个声明了类型但没有显式字段名的字段称为嵌入式字段。嵌入式字段必须指定为类型名T或指向非接口类型名*T的指针，T本身不能是指针类型。非限定的类型名作为字段名。

```go
// A struct with four embedded fields of types T1, *T2, P.T3 and *P.T4
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
	x, y int  // field names are x and y
}
```

以下声明是非法的，因为字段名在结构类型中必须是唯一的。

```go
struct {
	T     // conflicts with embedded field *T and *P.T
	*T    // conflicts with embedded field T and *P.T
	*P.T  // conflicts with embedded field T and *T
}
```

如果x.f是表示该字段或方法f的合法选择器，那么结构体x中嵌入字段的字段或方法f被称为promoted（提升）。

被提升的字段与结构体的普通字段一样，只是它们不能在结构体的复合字面量中作为字段名使用。

给定一个结构类型S和一个已定义类型T，被提升的方法被包含在结构体的方法集中，具体如下：

- 如果S包含一个嵌入字段T，那么S和`*S`的方法集都包含有接收者T的提升方法。*S的方法集还包括具有接收者`*T`的提升方法。
- 如果S包含一个内嵌字段`*T`，那么S和`*S`的方法集都包含有接收者T或`*T`的被提升方法。

字段声明后面可以有一个可选的字符串字面量标签（tag），它成为对应字段声明中所有字段的属性。空的标签字符串相当于一个不存在的标签。标签通过反射接口变得可见，并参与结构体的类型识别，但在其他方面被忽略。

```go
struct {
	x, y float64 ""  // an empty tag string is like an absent tag
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// A struct corresponding to a TimeStamp protocol buffer.
// The tag strings define the protocol buffer field numbers;
// they follow the convention outlined by the reflect package.
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```



### 参考资料

- [Dig101:Go 之聊聊 struct 的内存对齐](https://gocn.vip/topics/9759)
- [Go 之如何操作结构体的非导出字段](https://gocn.vip/topics/10553)
- https://gobyexample-cn.github.io/structs