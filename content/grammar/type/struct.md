# 结构体

结构体是字段的集合。

## 基本语法

### 类型定义

结构体定义的语法：

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

