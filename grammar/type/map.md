# Map

创建Map

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

## 修改 map

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

