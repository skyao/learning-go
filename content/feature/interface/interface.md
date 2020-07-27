# 接口

## 接口定义和实现

接口类型是由一组方法定义的集合。

```go
// 定义一个interface和它的方法
type Abser interface {
	Abs() float64
}

type Vertex struct {
	X, Y float64
}

// 让结构体实现interface要求的方法
func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

接口类型的值可以存放实现这些方法的任何值。

```go
var a Abser
v := Vertex{3, 4}
a = &v // *Vertex 实现了 Abser
```

## 隐式接口

类型通过实现那些方法来实现接口。 没有显式声明的必要；所以也就没有关键字“implements“。

隐式接口解藕了实现接口的包和定义接口的包：互不依赖。

因此，也就无需在每一个实现上增加新的接口名称。

常见的接口：

1.  [`fmt`](https://golang.org/pkg/fmt/) 包中定义的 [`Stringer`](https://golang.org/pkg/fmt/#Stringer)

   ```go
   type Stringer interface {
       String() string
   }
   ```

   `Stringer` 是一个可以用字符串描述自己的类型。`fmt`包 （还有许多其他包）使用这个来进行输出。

   ```go
   type Person struct {
   	Name string
   	Age  int
   }

   func (p Person) String() string {
   	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
   }
   ```

## 任意接口类型

在go中，如果要表示类型为任意类型，包括基础类型，可以这样做：

```go
type Value struct {
	v interface{}
}
```

有点类似java中的Object，但是go没有对象继承，也没有Object这种单根继承的root对象，为了表示所有类型，就需要使用interface关键字，而interface在go中是关键字，不是类型，因此要加`{}` 后缀。这个语法相对java有点特别。

