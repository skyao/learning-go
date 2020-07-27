---
date: 2020-07-04T23:50:00+08:00
title: Map类型
weight: 480
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的Map类型"
---

## Map

> 备注：内容摘录自 go语言实战

map 在使用之前必须用 `make` 而不是 `new` 来创建；值为 `nil` 的 map 是空的，并且不能赋值。

```go
m := make(map[string]string) // 语法是 "map[key的类型]value的类型"
m["key1"] = "value1"
fmt.Println(m["key1"])
```

或者直接通过指定key、value来创建：

```go
var m = map[string]string{
	"key1": "value1",
	"key2": "value2",
	"key3": "value3",	// 注意最后一行的结尾也必须有逗号
}
```

value的类型可以忽略，比如下面这种写法：

```go
var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

```

可以简化为：

```go
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

修改 map：

```go
func main() {
	m := make(map[string]int)

	m["Answer"] = 42	//在 map 中插入或修改
	fmt.Println("The value:", m["Answer"])  // 通过key获取值

	m["Answer"] = 48 //在 map 中插入或修改
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer") // 从 map 中删除key
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"] // 双赋值检测某个键存在，如果存在则第二个参数返回true
	fmt.Println("The value:", v, "Present?", ok)
}
```

## Map types

https://golang.org/ref/spec#Map_types

Map是由一种类型（称为元素类型）的元素组成的无序组，由另一种类型的一组唯一键（称为键类型）索引。未初始化的map的值为nil。

```
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = Type .
```

比较运算符 == 和 != 必须为键类型的操作数完全定义；因此键类型不能是函数、映射或分片。如果键类型是接口类型，必须为动态键值定义这些比较运算符，否则会引起运行时的恐慌。

```go
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

Map元素的数量称为其长度。对于map m来说，它可以通过内置函数len来发现，并且在执行过程中可能会改变。在执行过程中可以使用赋值来添加元素，并使用索引表达式来检索；可以使用内置的 delete 函数来删除元素。

使用内置函数make制作一个新的、空的map值，该函数将map类型和一个可选的容量提示作为参数。

```go
make(map[string]int)
make(map[string]int, 100)
```

初始容量并不约束其大小：为了容纳其中存储的项目数量map可以增长，但 nil map除外。nil map相当于一个空map，但不能添加任何元素。

## Maps

https://golang.org/doc/effective_go.html#maps

Map是一种方便而强大的内置数据结构，它将一种类型的值（键）与另一种类型的值（元素或值）关联起来。键可以是任何定义了 equality 运算符的类型，如整数、浮点和复数、字符串、指针、接口（只要动态类型支持equality）、结构体和数组。切片不能用作映射键，因为在它们上面没有定义equality。和切片一样，map 也持有对底层数据结构的引用。如果你把一个map传给一个函数，该函数改变了map的内容，那么这些改变将在调用者中可见。

Map可以使用通常的复合文字语法和冒号分隔的键值对来构建，所以在初始化时很容易构建它们。

```
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

分配和获取映射值在语法上看起来就像对数组和切片做同样的事情一样，只是索引不需要是一个整数。

```go
offset := timeZone["EST"]
```

试图用map中不存在的键来获取一个map值，将返回map中条目类型的0值。例如，如果map中包含整数，查找一个不存在的键将返回0。集合可以实现为一个值类型为bool的map。将map条目设置为true，将值放入集合中，然后通过简单的索引进行测试。

```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

有时你需要区分缺失的条目和零值。是否有 "UTC "的条目，还是因为map中根本没有这个条目而为0？你可以用一种多重赋值的形式来区分。

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

出于明显的原因，这被称为 "逗号 ok "习惯用法。在这个例子中，如果tz存在，秒数将被适当设置，ok将为true；如果不存在，秒数将被设置为0，ok将为false。这里有一个函数，把它和一个漂亮的错误报告放在一起。

```go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

为了测试map中是否存在，而不用担心实际值，你可以使用空白标识符(_)来代替通常的变量的值。

```go
_, present := timeZone[tz]
```

要删除一个map条目，使用delete内置函数，其参数是map和要删除的键。即使key已经不在map上，这样做也是安全的。

```go
delete(timeZone, "PDT")  // Now on Standard Time
```

### 参考资料

- [Dig101:Go 之读懂 map 的底层设计](https://gocn.vip/topics/9594)