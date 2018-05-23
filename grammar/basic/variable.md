# 变量

## 变量定义

`var` 语句定义一个变量的列表；跟函数的参数列表一样，类型在后面。

`var` 语句可以定义在包或函数级别。

```go
var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```

## 变量初始化

```go
// 变量定义可以包含初始值，每个变量对应一个。
var i, j int = 1, 2

func main() {
    // 如果初始化是使用表达式，则可以省略类型；
    // 变量从初始值中获得类型。
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```

