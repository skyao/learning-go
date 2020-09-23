---
date: 2020-07-04T23:50:00+08:00
title: package文档
weight: 2011
menu:
  main:
    parent: "test-gotesting"
description : ""
---

> 备注：https://golang.org/pkg/testing/



### 概述

Package testing 为Go package的自动测试提供支持。它旨在与 "go test "命令协同使用，该命令可以自动执行任何如下形式的函数：

```go
func TestXxx(*testing.T)
```

其中Xxx不以小写字母开头。函数名称的作用是识别测试routine。

在这些函数中，使用Error、Fail或相关方法来发出失败信号。

要编写一个新的测试套件，创建一个以 _test.go 结尾的文件，它包含了这里描述的 TestXxx 函数。把这个文件和被测试的包放在同一个包里。这个文件将被排除在常规的软件包构建之外，但是当运行 "go test" 命令时，这个文件将被包含在内。更多细节，请运行 "go help test "和 "go help testflag"。

一个简单的测试函数是这样的：

```go
func TestAbs(t *testing.T) {
    got := Abs(-1)
    if got != 1 {
        t.Errorf("Abs(-1) = %d; want 1", got)
    }
}
```

### Benchmarks

这种形式的函数：

```
func BenchmarkXxx(*testing.B)
```

被视为基准（ benchmarks）, 并在提供 -bench 标志时由 "go test "命令执行。基准是按顺序运行的。

testing flags的描述, 请见 https://golang.org/cmd/go/#hdr-Testing_flags

基准函数的例子是这样的。

```go
func BenchmarkRandInt(b *testing.B) {
    for i := 0; i < b.N; i++ {
        rand.Int()
    }
}
```

基准函数必须运行目标代码 b.N 次。在基准执行过程中，b.N会被调整，直到基准函数持续足够长的时间来可靠地计时。输出

```
BenchmarkRandInt-8   	68453040	        17.8 ns/op
```

意味着循环运行68453040次，每次循环速度为17.8ns。

如果基准在运行前需要一些昂贵的设置，计时器可能需要被重置：

```go
func BenchmarkBigLen(b *testing.B) {
    big := NewBig()
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        big.Len()
    }
}
```

如果基准需要测试并行环境下的性能，它可以使用RunParallel辅助函数；这样的基准应该与go test -cpu 标志一起使用:

```go
func BenchmarkTemplateParallel(b *testing.B) {
    templ := template.Must(template.New("test").Parse("Hello, {{.}}!"))
    b.RunParallel(func(pb *testing.PB) {
        var buf bytes.Buffer
        for pb.Next() {
            buf.Reset()
            templ.Execute(&buf, "World")
        }
    })
}
```

### Examples

该软件包还可以运行和验证example代码。example函数可能包含一个以 "Output: "开头的结尾行注释，并在测试运行时与函数的标准输出进行比较。(比较会忽略前导空格和后导空格。)这些都是example的例子。

```go
func ExampleHello() {
    fmt.Println("hello")
    // Output: hello
}

func ExampleSalutations() {
    fmt.Println("hello, and")
    fmt.Println("goodbye")
    // Output:
    // hello, and
    // goodbye
}
```

注释前缀 "Unordered output:" 和 "Output:" 一样，但匹配任何行序。

```go
func ExamplePerm() {
    for _, value := range Perm(5) {
        fmt.Println(value)
    }
    // Unordered output: 4
    // 2
    // 1
    // 3
    // 0
}
```

没有输出注释的Example函数会被编译但不会被执行。

为包、函数F、类型T和类型T上的方法M声明例子的命名惯例是：

```go
func Example() { ... }
func ExampleF() { ... }
func ExampleT() { ... }
func ExampleT_M() { ... }
```

包/类型/函数/方法的多个example函数，可以通过在名称后添加一个不同的后缀来提供。后缀必须以小写字母开头。

```go
func Example_suffix() { ... }
func ExampleF_suffix() { ... }
func ExampleT_suffix() { ... }
func ExampleT_M_suffix() { ... }
```

当整个测试文件包含一个example函数，至少一个其他函数、类型、变量或常量声明，并且没有测试或基准函数时，整个测试文件将作为example呈现。

### Skipping

在运行时可以通过调用 `*T` 或 `*B` 的Skip方法来跳过测试或基准。

```
func TestTimeConsuming(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping test in short mode.")
    }
    ...
}
```

### Subtests and Sub-benchmarks

T和B的Run方法允许定义子测试(subtest)和子基准(sub-benchmark)，而不必为每个子基准定义单独的函数。这使得像表格驱动(table-driven)的基准和创建分层测试这样的用途成为可能。它还提供了一种共享通用设置和拆卸代码的方法。

```go
func TestFoo(t *testing.T) {
    // <setup code>
    t.Run("A=1", func(t *testing.T) { ... })
    t.Run("A=2", func(t *testing.T) { ... })
    t.Run("B=1", func(t *testing.T) { ... })
    // <tear-down code>
}
```

每个子测试和子基准都有一个唯一名称：顶层测试的名称和传递给Run的名称序列的组合，用斜线隔开，并有一个可选的尾部序列号，用于消除歧义。

`-run` 和 `-bench` 命令行标志的参数是一个与测试名称相匹配的 unanchored 正则表达式。对于有多个斜线分隔元素的测试，比如子测试，参数本身是斜线分隔的，表达式依次匹配每个名称元素。因为它是unanchored，所以空的表达式可以匹配任何字符串。例如，用 "matching "来表示 "whose name contains"。

```bash
go test -run ''      # Run all tests.
go test -run Foo     # Run top-level tests matching "Foo", such as "TestFooBar".
go test -run Foo/A=  # For top-level tests matching "Foo", run subtests matching "A=".
go test -run /A=1    # For all top-level tests, run subtests matching "A=1".
```

子测试也可以用来控制并行性。父测试只有在它的所有子测试完成后才会完成。在这个例子中，所有的测试都是相互并行运行的，而且只允许相互并行运行，而不考虑可能定义的其他顶层测试。

```go
func TestGroupedParallel(t *testing.T) {
    for _, tc := range tests {
        tc := tc // capture range variable
        t.Run(tc.Name, func(t *testing.T) {
            t.Parallel()
            ...
        })
    }
}
```

Race检测器会在程序超过8192个并发goroutines的情况下杀死程序，所以在设置 -race 标志的情况下运行并行测试时要小心。

在并行子测试完成之前，Run不会返回，提供了一种在一组并行测试后进行清理的方法。

```go
func TestTeardownParallel(t *testing.T) {
    // This Run will not return until the parallel tests finish.
    t.Run("group", func(t *testing.T) {
        t.Run("Test1", parallelTest1)
        t.Run("Test2", parallelTest2)
        t.Run("Test3", parallelTest3)
    })
    // <tear-down code>
}
```

### Main

有时，测试程序需要在测试前或测试后进行额外的 setup 或 teardown。有时，测试也需要控制哪些代码在主线程上运行。为了支持这些和其他情况，如果一个测试文件包含一个函数：

```go
func TestMain(m *testing.M)
```

然后生成的测试将调用 TestMain(m) 而不是直接运行测试。TestMain 在主 goroutine 中运行，并且可以围绕对 m.Run 的调用进行任何必要的设置和拆卸。m.Run 将返回一个退出代码，这个代码可以传递给 os.Exit。如果 TestMain 返回，测试包装器将把 m.Run 的结果传递给 os.Exit。

当TestMain被调用时，flag.Parse还没有被运行。如果TestMain依赖于命令行标志，包括测试包的标志，它应该明确地调用flag.Parse。命令行标志总是在测试或基准函数运行时被解析。

TestMain的简单实现如下：

```
func TestMain(m *testing.M) {
	// call flag.Parse() here if TestMain uses flags
	os.Exit(m.Run())
}
```


