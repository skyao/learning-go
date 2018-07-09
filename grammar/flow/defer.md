# defer语句

## 使用场景

defer 语句会延迟函数的执行直到外层函数返回，通常用于执行清理操作。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

其他使用场景，如释放mutex：

```go
mu.Lock()
defer mu.Unlock()
```

打印footer：

```go
printHeader()
defer printFooter()
printContent1()
printContent2()
```

## 使用事项

1. 调用所需的参数会立刻生成

   ```go
   func a() {
       i := 0
       defer fmt.Println(i)
       i++
       return
   }
   ```

   这里会打印0，因为 `defer fmt.Println(i)` 执行时i为0，参数在此时确定，后面的改动不会影响defer语句的参数。

2. 多个defer调用会入栈，后进先出

   ```go
   func b() {
       for i := 0; i < 4; i++ {
           defer fmt.Print(i)
       }
   }
   ```

   这里会打印3210，顺序和defer语句的顺序相反。

3. defer语句有机会修改函数返回值

   ```go
   func c() (i int) {
       defer func() { i++ }()
       return 1
   }
   ```

   这里的函数返回值会被defer修改，从而返回2。

## 参考资料

- [Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover)