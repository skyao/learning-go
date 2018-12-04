# switch语句

switch的语法和for、if类似，同样的括号和花括号使用规则，同样的容许在switch前执行一个简单的语句，同样的变量访问方位限制：

```go
switch os := runtime.GOOS; os {
    case "darwin":
    	fmt.Println("OS X.")
    case "linux":
    	fmt.Println("Linux.")
    default:
    	fmt.Printf("%s.", os)
}
```

特别需要支出的是，和c、java中的switch语句不同，golang中的switch在命中某个case子语句并执行完成之后，会自动终结分支并结束switch语句。这是默认行为，和c，java中会自动继续下一个分支匹配，需要明确break才能退出不同。

如果想继续执行后面的case子语句，需要在case子语句最后使用 `fallthrough` 语句。

## 执行顺序

switch 的条件从上到下顺序执行，当匹配成功的时候终止。

```go
switch i {
    case 0:
    case f():
}
```

当 `i==0` 时不会调用 `f`。

## if变体

没有条件的 switch 等同于 `switch true` ，这个变体可以用更清晰的形式来编写多个判断条件的 if 语句：

```go
t := time.Now()
// 等同于 switch true {
switch {
    case t.Hour() < 12:
    	fmt.Println("Good morning!")
    case t.Hour() < 17:
    	fmt.Println("Good afternoon.")
    default:
    	fmt.Println("Good evening.")
}
```

