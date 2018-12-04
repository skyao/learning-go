# 循环

Go 只有一种循环结构 `for` 循环。

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

和Java的语法相反：

1. for 后面没有括号`()`，注意是强制一定不能有
2. 循环体必须有`{}`

跟 C 或者 Java 中一样，可以让前置、后置语句为空：

```go
sum := 1
for ; sum < 1000; {
    sum += sum
}
```

这就非常类似C、Java中的while循环了，因此干脆继续简写，省略掉分括号：

```go
sum := 1
for sum < 1000 {
    sum += sum
}
```

更绝一点，死循环：

```go
for {
}
```

