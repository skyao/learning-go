# select语句

select是Golang中的控制语句，语法类似于switch语句。但是select只用于通信，要求每个case必须是IO操作。

不带default语句的select会阻塞直到某个case满足：

```go
ch1 := make (chan int, 1)
ch2 := make (chan int, 1)

select {
case <-ch1:
    fmt.Println("ch1 pop one element")
case <-ch2:
    fmt.Println("ch2 pop one element")
}
```

如果两个case同时满足，则随机执行某个case语句，其他case语句不会执行。 

如果不想阻塞，则可以带上default子语句：

```go
select {
case <-ch1:
    fmt.Println("ch1 pop one element")
case <-ch2:
    fmt.Println("ch2 pop one element")
default:
    fmt.Println("not ready yet")
}
```

如果两个case条件都不满足，则直接调到default流程而不阻塞。

