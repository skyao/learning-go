# 方法

## 方法定义

Go 没有类。但是可以在结构体类型上定义方法。

```go
type Vertex struct {
	X, Y float64
}

// 方法接收者 出现在 func 关键字和方法名之间
func (v *Vertex) Abs() float64 {	// 注意的方法接受者是指针类型
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := &Vertex{3, 4}
	fmt.Println(v.Abs())
}
```

你可以对包中的 *任意* 类型定义任意方法，而不仅仅是针对结构体。

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {	// 注意的方法接受者是值类型
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}
```

注意：不能对来自其他包的类型或基础类型定义方法。

## 指针接受者

有两个原因需要使用指针接收者：

1. 避免在每个方法调用中拷贝值（如果值类型是大的结构体的话会更有效率）
2. 方法可以修改接收者指向的值

```go
type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) { 
    // 如果这里不是指针类型而是值类型，而传入的是原始结构体的一个复制后的副本
    // 修改副本是不会影响原始结构体的数据的
    // 而且会有复制结构体的开销
	v.X = v.X * f
	v.Y = v.Y * f
}

func (v *Vertex) Abs() float64 {
    // 如果这里不是指针类型而是值类型，而传入的是原始结构体的一个复制后的副本
    // 因为是只读，读取原始结构体和副本，数值都相同，不影响计算结果
    // 但是会有复制结构体的开销
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := &Vertex{3, 4}
	v.Scale(5)
	fmt.Println(v, v.Abs())
}
```

