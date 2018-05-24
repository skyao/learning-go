# 常量

## 常量的定义

常量的定义与变量类似，只不过使用 `const` 关键字。

```go
const Pi = 3.14

func main() {
	const World = "world"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go rules?", Truth)
}
```

常量可以是字符、字符串、布尔或数字类型的值。

注意：常量不能使用 `:=` 语法定义。

## 数值常量

数值常量是高精度的 **值**。

```go
const (
	Big   = 1 << 100
	Small = Big >> 99
)
```

一个未指定类型的常量由上下文来决定其类型。