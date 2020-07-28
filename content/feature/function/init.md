---
date: 2020-07-04T23:50:00+08:00
title: init函数
weight: 811
menu:
  main:
    parent: "feature-function"
description : "go语言的init函数"
---

## Effective Go

https://golang.org/doc/effective_go.html#init

最后，每个源文件都可以定义自己的niladic init函数来设置任何需要的状态。(其实每个文件都可以有多个init函数。) 而finally的意思是：init是在包中的所有变量声明都评估了初始化器之后才被调用的，而那些初始化器只有在所有导入的包都有初始化器之后才被评估。(其实每个文件都可以有多个init函数。)而finally的意思是最后：init是在包中所有的变量声明都评估了它们的初始化器之后才被调用的，而这些初始化器只有在所有的导入包都被初始化之后才被评估。

除了不能用声明来表达的初始化之外，init函数的一个常见用途是在真正执行开始之前验证或修复程序状态的正确性。

```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## 参考资料

### 五分钟理解golang的init函数

https://zhuanlan.zhihu.com/p/34211611

**init函数的主要作用：**

- 初始化不能采用初始化表达式初始化的变量。
- 程序运行前的注册。
- 实现 sync.Once 功能。
- 其他

**init函数的主要特点：**

- init函数先于main函数自动执行，不能被其他函数调用；
- init函数没有输入参数、返回值；
- 每个包可以有多个init函数；
- **包的每个源文件也可以有多个init函数**，这点比较特殊；
- 同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序。
- 不同包的init函数按照包导入的依赖关系决定执行顺序。

**golang程序初始化：**

golang程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：

1. 初始化导入的包（包的初始化顺序并不是按导入顺序（“从上到下”）执行的，runtime需要解析包依赖关系，没有依赖的包最先初始化，与变量初始化依赖关系类似
2. 初始化包作用域的变量（该作用域的变量的初始化也并非按照“从上到下、从左到右”的顺序，runtime解析变量依赖关系，没有依赖的变量最先初始化）；
3. 执行包的init函数；

几个值得注意的地方：

- 初始化顺序：**变量初始化 -> init() -> main()**
- 同一个包不同源文件的init函数执行顺序，golang spec没做说明，以上述程序输出来看，执行顺序是源文件名称的字典序。
- init函数不可以被调用，上面代码会提示：undefined: init
- init函数比较特殊，可以在包里被多次定义。
- init函数的主要用途：初始化不能使用初始化表达式初始化的变量
- golang对没有使用的导入包会编译报错，但是有时我们只想调用该包的init函数，不使用包导出的变量或者方法： 
	* `import _ "net/http/pprof"`



