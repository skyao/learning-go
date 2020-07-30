---
date: 2020-07-04T23:50:00+08:00
title: 闭包
weight: 816
menu:
  main:
    parent: "feature-function"
description : "go语言的闭包"
---

闭包相关的经典总结：

- **闭包=函数+引用环境**

- **对象是附有行为的数据，而闭包是附有数据的行为**



https://gobyexample-cn.github.io/closures

Go 支持匿名函数， 并能用其构造闭包。 匿名函数在你想定义一个不需要命名的内联函数时是很实用的。

`intSeq` 函数返回一个在其函数体内定义的匿名函数。 返回的函数使用闭包的方式 *隐藏* 变量 `i`。 返回的函数 *隐藏* 变量 `i` 以形成闭包。

```go
func intSeq() func() int {
    i := 0
    return func() int {
        i++
        return i
    }
}
```

我们调用 `intSeq` 函数，将返回值（一个函数）赋给 `nextInt`。 这个函数的值包含了自己的值 `i`，这样在每次调用 `nextInt` 时，都会更新 `i` 的值。

```go
func main() {

   nextInt := intSeq()

   // 通过多次调用 nextInt 来看看闭包的效果：
   fmt.Println(nextInt())  // 1
   fmt.Println(nextInt())  // 2
   fmt.Println(nextInt())  // 3

   // 为了确认这个状态对于这个特定的函数是唯一的，我们重新创建并测试一下。
   newInt2 := intSeq()
   fmt.Println(newInt2())  // 1
}
```






### 参考资料

- https://gobyexample-cn.github.io/closures
- [Closure in Golang](https://www.jianshu.com/p/3934e62d78a1)： 推荐
- [Closures in Go序列](https://www.calhoun.io/closures-in-go/)： 强烈推荐

