# If语句

类似for循环，go中的if语句也是同样，if后面不能有括号，而if里面必须要有花括号：

```go
if x < 0 {
    return sqrt(-x) + "i"
}
```

跟 `for` 一样，`if` 语句可以在条件之前执行一个简单的语句：

```go
if v := math.Pow(x, n); v < lim {
    // v在这里可以访问
    return v
}
// v在这里不可以访问
```

注意：这个语句定义的变量的作用域仅在 `if` 范围之内，包括else：

```go
if v := math.Pow(x, n); v < lim {
    return v
} else {
    // else这里可以访问
    fmt.Printf("%g >= %g\n", v, lim)
}
```

