# 语句

## 循环语句

Go 只有一种循环结构：`for` 循环。

```go
func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```

基本的 `for` 循环除了没有了 `( )` 之外（甚至强制不能使用它们），看起来跟 C 或者 Java 中做的一样，而 `{ }` 是必须的。

跟 C 或者 Java 中一样，可以让前置、后置语句为空。

```go
func main() {
	sum := 1
	for ; sum < 1000; {
		sum += sum
	}
	fmt.Println(sum)
}
```

可以省略分号：

```go
func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```

等同于其他语言的while（）循环。

如果省略了循环条件，循环就不会结束，因此可以用更简洁地形式表达死循环。

```go
for {
}
```

## if语句

`if` 语句除了没有了 `( )` 之外（甚至强制不能使用它们），看起来跟 C 或者 Java 中的一样，而 `{ }` 是必须的。

```go
func sqrt(x float64) string {
	if x < 0 {
		return sqrt(-x) + "i"
	}
	return fmt.Sprint(math.Sqrt(x))
}
```

跟 `for` 一样，`if` 语句可以在条件之前执行一个简单的语句。

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}
```

由这个语句定义的变量的作用域仅在 `if` 范围之内。

个人理解：和在if语句外面写一个var语句相比，可以将变量作用域限制在if语句的范围内。

在 `if` 的便捷语句定义的变量同样可以在任何对应的 `else` 块中使用。

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// 这里开始就不能使用 v 了
	return lim
}
```

## switch

Go的switch非常灵活，表达式不必是常量或整数，但是比较的类型必须一致。

执行的过程从上至下，直到找到匹配项。

```go
func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.", os)
	}
}
```

Go里面switch默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用fallthrough强制执行后面的case代码。

```go
    case "linux: //假设这里匹配
            fmt.Println("The integer was <= 4")
            fallthrough //会继续走下一个case
```

注意：fallthrough不能用在switch的最后一个分支。

而如果switch没有表达式，它会匹配true。 没有条件的 switch 同 `switch true` 一样。

```go
switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
```

这一构造使得可以用更清晰的形式来编写长的 if-then-else 链。在其他语言中写多个if else判断时一般格式如下：

```java
if () {   
} else if () {
} else if () {
} else if () {
}
```

用go的switch true简化格式的确方便。





