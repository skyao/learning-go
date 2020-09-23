---
date: 2020-07-04T23:50:00+08:00
title: go test flag参数
weight: 2012
menu:
  main:
    parent: "test-gotesting"
description : ""
---

> 备注：https://golang.org/cmd/go/#hdr-Testing_flags

'go test' 命令既接受适用于 'go test' 本身的标志，也接受适用于产生的测试二进制的标志。

其中有几个标志可以控制profiling，并写出适合 "go tool pprof "的执行profiling；运行 "go tool pprof -h "可以获得更多信息。pprof 的 --alloc_space, --alloc_objects, 和 --show_bytes 选项控制了信息的呈现方式。

## go test识别的标志

下面的标志被 'go test' 命令所识别，并控制任何测试的执行。

### `-bench regexp`

只运行与正则表达式匹配的基准benchmarks。

默认情况下，不运行任何基准。

要运行所有基准，请使用 '-bench .' 或 '-bench=.'。

正则表达式由无括号的斜杠(/)字符分割成正则表达式序列，如果有的话，基准的标识符的每个部分必须与序列中的相应元素匹配。

> Possible parents of matches are run with b.N=1 to identify sub-benchmarks. For example,  given -bench=X/Y, top-level benchmarks matching X are run with b.N=1 to find any sub-benchmarks matching Y, which are then run in full.

在b.N=1的情况下，运行可能的匹配parent，以确定子基准。例如，给定-bench=X/Y，用b.N=1运行与X匹配的顶层基准，找出与Y匹配的子基准，然后完整地运行。

### `-benchtime t`

为每个基准运行足够的迭代，花费时间t，指定为 time.Duration（例如，-benchtime 1h30s）。

默认为1秒（1s）。

特殊语法Nx表示运行基准N次（例如，-benchtime 100x）。

### `-count n`

运行每个测试和基准 n 次（默认 1）。

如果设置了 -cpu，则对每个 GOMAXPROCS 值运行 n 次。

示例总是运行一次。

### `-cover`

启用覆盖率分析。

请注意，由于覆盖率是通过在编译前对源代码进行注释来实现的，因此在启用覆盖率后，编译和测试失败时可能会报告与原始源代码不一致的行数。

### `-covermode set,count,atomic`

设置正在测试的软件包的覆盖率分析模式，默认为 "set"，除非启用 -race，这种情况下默认为 "atomic"。

The values:

- set: bool: does this statement run?
- count: int: how many times does this statement run?
- atomic: int: count, but correct in multithreaded tests; significantly more expensive.

设置 -cover.

### `-coverpkg pattern1,pattern2,pattern3`

在每个测试中对匹配模式的软件包应用覆盖率分析。

默认情况下，每个测试只分析被测试的包。

请参阅 'go help packages' 以了解软件包模式的描述。

设置 -cover.

### `-cpu 1,2,4`

指定执行测试或基准的 GOMAXPROCS 值的列表。默认为GOMAXPROCS的当前值。

### `-failfast`

第一次测试失败后，不要再开始新的测试。

### `-list regexp`

列出与正则表达式匹配的测试、基准或示例。

不会运行任何测试、基准或示例。这将只列出顶层测试，不会显示子测试或子基准。不显示子测试或子基准。

### `-parallel n`

允许并行执行调用 t.Parallel 的测试函数。

这个标志的值是同时运行的测试的最大数量；默认情况下，它被设置为GOMAXPROCS的值。

请注意，-parallel只适用于单个测试二进制文件中。

根据 -p 标志的设置， 'go test' 命令也可以并行运行不同包的测试。 (见'go help build')。

### `-run regexp`

只运行那些与正则表达式相匹配的测试和示例。

对于测试来说，正则表达式被无括号的斜杠（/）字符分割成正则表达式序列，如果有的话，测试的标识符的每一部分必须与序列中的相应元素相匹配。

> Note that possible parents of matches are run too, so that -run=X/Y matches and runs and reports the result of all tests matching X, even those without sub-tests matching Y, because it must run them to look for those sub-tests.

注意，匹配的可能的parent也会被运行，所以-run=X/Y匹配并运行和报告所有匹配X的测试的结果，甚至那些没有子测试匹配Y的测试，因为它必须运行它们来寻找这些子测试。

### `-short`

告诉长期运行的测试缩短其运行时间。

默认情况下是关闭的，但在 all.bash 期间进行设置，这样安装Go树时就可以运行理智性检查，但不会花时间运行详尽的测试。

### `-timeout d`

如果一个测试二进制文件运行时间超过持续时间d，就会产生恐慌/panic。

### `-v`

Verbose输出：在测试运行时记录所有测试。即使测试成功，也会打印所有来自Log和Logf调用的文本。

### `-vet list`

配置在 "go test "期间对 "go vet " 的调用，以使用逗号分隔的vet检查列表。

如果列表为空，"go test "就会使用一个被认为总是值得处理的精选检查列表来运行 "go vet"。

如果列表为 "off"，"go test "根本就不运行 "go vet"。

## 描述测试的标记

下面的标志也是'go test'所能识别的，并且可以在执行过程中用来描述测试。

### `-benchmem`

为基准测试打印内存分配统计数据。

### `-blockprofile block.out`

当所有测试完成后，将 goroutine blocking profile写入指定文件。

像 -c 那样写入测试二进制文件。

### `-blockprofilerate n`

通过携带 n 调用 runtime.SetBlockProfileRate 来控制goroutine阻塞配置文件中提供的细节。

参见'go doc runtime.SetBlockProfileRate'。

profiler的目标是，平均每 n 纳秒对程序被阻塞的时间进行一次阻塞事件采样。默认情况下，如果设置了-test.blockprofile而没有这个标志，则会记录所有阻塞事件，相当于-test.blockprofilerate=1。

> The profiler aims to sample, on average, one blocking event every n nanoseconds the program spends blocked. By default, if -test.blockprofile is set without this flag, all blocking events are recorded, equivalent to -test.blockprofilerate=1.

### `-coverprofile cover.out`

    Write a coverage profile to the file after all tests have passed.
    Sets -cover.

### `-cpuprofile cpu.out`

Write a CPU profile to the specified file before exiting.
    Writes test binary as -c would.

### `-memprofile mem.out`

Write an allocation profile to the file after all tests have passed.
    Writes test binary as -c would.

### `-memprofilerate n`

Enable more precise (and expensive) memory allocation profiles by setting runtime.MemProfileRate. See 'go doc runtime.MemProfileRate'.

To profile all memory allocations, use -test.memprofilerate=1.

### `-mutexprofile mutex.out`

Write a mutex contention profile to the specified file

when all tests are complete.

Writes test binary as -c would.

### `-mutexprofilefraction n`

Sample 1 in n stack traces of goroutines holding a contended mutex.

### `-outputdir directory`

将profiling的输出文件放置在指定的目录中，默认是 "go test "运行的目录。

### `-trace trace.out`

在退出前向指定文件写入执行跟踪。



## 其他使用细节

这些标志中的每一个都可以用可选的 'test.' 前缀来识别，就像 -test.v 一样。然而，当直接调用生成的测试二进制文件 ('go test -c'的结果)时，前缀是必须的。

在调用测试二进制文件之前，'go test'命令会适当地重写或删除可选包列表之前和之后的识别标志。

例如，该命令

```bash
go test -v -myflag testdata -cpuprofile=prof.out -x
```

将编译测试二进制文件，然后以

```bash
pkg.test -test.v -myflag testdata -test.cpuprofile=prof.out
```

运行。

(-x 标志被删除了，因为它只适用于 go 命令的执行，而不是测试本身。)

生成profile文件的test flag (除了覆盖率) 也会将测试二进制文件留在 pkg.test 中，供分析profile文件时使用。

当 'go test' 运行一个测试二进制文件时，它是在相应软件包的源代码目录下进行的。根据测试的不同，当直接调用生成的测试二进制文件时，可能需要做同样的工作。

命令行包列表，如果存在的话，必须出现在任何不为go测试命令所知的标志之前。继续上面的例子，软件包列表必须出现在-myflag之前，但也可以出现在-v的任何一边。

当'go test'以包列表模式运行时，'go test'会缓存成功的包测试结果，以避免不必要的重复运行测试。要禁用测试缓存，请使用除可 cacheable 标志以外的任何测试标志或参数。明确禁止测试缓存的习惯方法是使用 -count=1。

为了不让测试二进制文件的参数被解释为已知的标志或包名，使用 -args (参见 'go help test')，它将命令行的其余部分不加解释和修改地传递给测试二进制文件。

例如，命令

```bash
go test -v -args -x -v
```

将编译测试二进制文件，然后以

```bash
pkg.test -test.v -x -v
```

运行。类似的

```bash
go test -args math
```

将编译测试二进制文件，然后以

```bash
pkg.test math
```

运行。

在第一个例子中，-x和第二个-v被传递到测试二进制文件中，没有变化，对go命令本身没有影响。在第二个例子中，参数math被传递到测试二进制文件中，而不是解释为包列表。