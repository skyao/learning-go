# Panic和Recover

**Panic** 是一种内置函数，可以阻止普通的控制流并开始*panicking*。 当函数g调用panic时，g的执行停止，g中的任何defer函数都正常执行，然后f返回其调用者f。对于调用者，f然后表现得像是对panic的调用。该过程继续向上移动，直到当前goroutine中的所有函数都返回，此时程序（备注：不是线程或者gorutine，而是进程）崩溃。可以通过直接调用 panic 来启动panic。也可能由运行时错误引起，例如越界数组访问。

Recover是一个内置函数，可以重新控制panic的goroutine。Recover仅在defer函数内有用。在正常执行期间，对recover的调用将返回nil并且没有其他效果。如果当前goroutine处于panic状态，则对recover的调用将获取赋值给panic的值并恢复正常执行。

```go
func main() {
	f()
	fmt.Println("Returned normally from f.")
}

func f() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered in f", r)
		}
	}()
	fmt.Println("Calling g.")
	g(0)
	fmt.Println("Returned normally from g.")
}

func g(i int) {
	if i > 3 {
		fmt.Println("Panicking!")
		panic(fmt.Sprintf("%v", i))
	}
	defer fmt.Println("Defer in g", i)
	fmt.Println("Printing in g", i)
	g(i + 1)
}
```



## 参考资料

- [Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover)