## 函数

在golang中，函数也是一种值，也可以赋值。

```go
hypot := func(x, y float64) float64 {
    return math.Sqrt(x*x + y*y)
}

fmt.Println(hypot(3, 4))
```

## 函数的闭包

Go 函数可以是闭包的。闭包是一个函数值，它来自函数体的外部的变量引用。 函数可以对这个引用值进行访问和赋值；换句话说这个函数被“绑定”在这个变量上。

```go
func adder() func(int) int {
	sum := 0 // 每个闭包都被绑定到其各自的 sum 变量上，这个 sum 变量是每个闭包独立的
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

例如，函数 `adder` 返回一个闭包。每个闭包都被绑定到其各自的 `sum` 变量上。