---
date: 2020-07-04T23:50:00+08:00
title: defer语句
weight: 611
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 defer 语句"
---

## go语言实战

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

使用事项：

1. 调用所需的参数会立刻评估

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

## golang语言规范

"defer"语句调用函数，该函数的执行被推迟到外围函数返回的那一刻，这可能是因为外围函数执行了一个返回语句，到达了其函数体的终点，或者是因为相应的goroutine发生panic/恐慌。

```
DeferStmt = "defer" Expression .
```

表达式必须是函数或方法调用，**不能用括号**。内置函数的调用与表达式语句一样受到限制。

每次执行 "defer"语句时，函数值和调用的参数都会像往常一样被评估并重新保存，但实际函数不会被调用。相反，deferred的函数在外围函数return之前立即被调用，其顺序与defer的顺序相反。也就是说，如果外围函数通过一个显式return语句返回，则在该return语句设置了任何结果参数之后，但在函数return给调用者之前，defer函数会被执行。如果一个defer函数的值评价为nil，则在函数被调用时，而不是在 "defer"语句被执行时，执行就会panic/慌乱。

例如，如果defer函数是一个函数字面量，而外围的函数有命名的结果参数，这些结果参数在字面量的范围内，那么defer函数可以在结果参数被返回之前访问和修改它们。如果defer函数有任何返回值，那么当函数完成时，它们将被丢弃。(也请参见关于处理恐慌的章节。)

```go
lock(l)
defer unlock(l)  // unlocking happens before surrounding function returns

// prints 3 2 1 0 before surrounding function returns
for i := 0; i <= 3; i++ {
	defer fmt.Print(i)
}

// f returns 42
func f() (result int) {
	defer func() {
		// result is accessed after it was set to 6 by the return statement
		result *= 7
	}()
	return 6
}
```

## Effective Go

https://golang.org/doc/effective_go.html#defer

Go 的 defer 语句安排在执行 defer 的函数返回之前立即运行函数调用（defer函数）。这是一种不寻常但有效的方法，用于处理一些情况，例如无论函数走哪条路径返回，都必须释放资源。规范的例子是解锁mutex或关闭文件。

```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

defer对Close这样的函数的调用有两个好处。首先，它保证你永远不会忘记关闭文件，如果你后来编辑函数添加了新的返回路径，就很容易犯这个错误。第二，它意味着close坐在open的附近，这比把它放在函数的最后要清楚得多。

defer函数的参数（如果函数是方法，则包括接收方）是在defer执行时评估的，而不是在调用执行时评估的。除了避免担心变量在函数执行时改变值之外，这意味着单个defer调用点可以defer多个函数的执行。下面是一个例子：

```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

defer函数是按照 LIFO 顺序执行的，所以这段代码会在函数返回时导致 4 3 2 1 0 被打印出来。更有价值的例子是通过程序跟踪函数执行的简单方法。我们可以写几个简单的跟踪例程，比如这样：

```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

我们可以更好地利用 defer 函数的参数在 defer 执行时被评估这一事实。追踪例程可以设置未追踪例程的参数。这个例子：

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

打印：

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

对于习惯了其他语言的块级资源管理的程序员来说，defer可能看起来很奇特，但它最有趣和最强大的应用恰恰来自于它不是基于块而是基于函数。

### 参考资料

- [Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover)