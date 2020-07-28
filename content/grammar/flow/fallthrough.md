---
date: 2020-07-04T23:50:00+08:00
title: fallthrough语句
weight: 610
menu:
  main:
    parent: "grammar-flow"
description : "go语言的 fallthrough 语句"
---

## golang语言规范

https://golang.org/ref/spec#Fallthrough_statements

"fallthrough"语句将控制权转移到 "switch" 语句中下一个case子句的第一条语句。它只能作为这种子句中的最后一个非空语句使用。


### 参考资料

- [go语言fallthrough的用法心得](https://www.cnblogs.com/zsy/p/6741902.html)： 
	* Go里面 switch 默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用fallthrough强制执行后面的case代码。
	* fallthrough不能用在switch的最后一个分支
	* fallthrough到下一个case块时，**不执行case匹配检查！不执行case匹配检查！不执行case匹配检查！**

特别注意最后一条，有点和常识不符合（我原本理解的fallthrough只是取消break，然后继续做下一个case的匹配，但实际fallthrough把case匹配检查也取消了）：

```go
switch {
case true:
   fmt.Println("The integer was <= 5")
   fallthrough
case false:
   fmt.Println("The integer was <= 6")
   fallthrough
default:
   fmt.Println("default case")
}
```

打印结果：

```bash
The integer was <= 5
The integer was <= 6
default case
```

