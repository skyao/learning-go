---
date: 2020-07-04T23:50:00+08:00
title: which语句
weight: 602
menu:
  main:
    parent: "grammar-flow"
description : "go语言类型中的 which 语句"
---


## switch语句

switch的语法和for、if类似，同样的括号和花括号使用规则，同样的容许在switch前执行一个简单的语句，同样的变量访问范围限制：

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

### 执行顺序

switch 的条件从上到下顺序执行，当匹配成功的时候终止。

```go
switch i {
    case 0:
    case f():
}
```

当 `i==0` 时不会调用 `f`。

### if变体

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

## Switch statements

https://golang.org/ref/spec#Switch_statements

"switch "语句提供多向执行。表达式或类型指定符与 "switch "内的 "case "进行比较，以确定执行哪个分支。

```
SwitchStmt = ExprSwitchStmt | TypeSwitchStmt .
```

有两种形式：表达式switch和类型switch。在表达式switch中，case包含表达式，与switch表达式的值进行比较。在类型switch中，case中包含的类型是与特别注释的switch表达式的类型进行比较的。switch表达式在一个switch语句中只被评估一次。

### 表达式switch

在表达式switch中，对switch表达式进行评估，并从左到右、从上到下评估case表达式，case表达式不一定是常量，第一个等于switch表达式的会触发执行相关case的语句，其他case则跳过。如果没有匹配的情况，并且有一个 "default"情况，则执行它的语句。最多只能有一个 default case，它可能出现在 "switch "语句的任何地方。缺少的switch表达式相当于布尔值true。

```
ExprSwitchStmt = "switch" [ SimpleStmt ";" ] [ Expression ] "{" { ExprCaseClause } "}" .
ExprCaseClause = ExprSwitchCase ":" StatementList .
ExprSwitchCase = "case" ExpressionList | "default" .
```

如果switch表达式的值是一个无类型常量，则首先隐式转换为其默认类型；如果是一个无类型布尔值，则首先隐式转换为布尔类型。预先声明的无类型值nil不能作为switch表达式使用。

如果一个case表达式是无类型的，它首先被隐式转换为switch表达式的类型。对于每个（可能转换的）case表达式x和switch表达式的值t，x == t必须是一个有效的比较。

换句话说，switch表达式被当作是用来声明和初始化一个没有显式类型的临时变量t；正是t的那个值与每个case表达式x进行了相等性测试。

在一个case或default子句中，最后一个非空语句可以是一个（可能被标记为）"fallthrough "语句，以表明控制权应该从这个子句的末尾流向下一个子句的第一个语句。否则控制流向 "switch "语句的末尾。"fallthrough "语句可以作为一个表达式switch的所有分句的最后一条语句出现，但最后一个分句除外。

switch表达式之前可以有一个简单的语句，该语句在表达式被评估之前执行。

```go
switch tag {
default: s3()
case 0, 1, 2, 3: s1()
case 4, 5, 6, 7: s2()
}

switch x := f(); {  // missing switch expression means "true"
case x < 0: return -x
default: return x
}

switch {
case x < y: f1()
case x < z: f2()
case x == 4: f3()
}
```

实现限制。编译器可能不允许多个case表达式求值于同一个常量。例如，目前的编译器不允许在case表达式中使用重复的整数、浮点或字符串常量。

### 类型switch

类型switch比较的是类型而不是值。在其他方面，它类似于表达式switch。它由一个特殊的switch表达式标记，它具有使用保留字类型而不是实际类型的类型断言形式。

```go
switch x.(type) {
// cases
}
```

然后，case将实际类型T与表达式x的动态类型进行匹配，与类型断言一样，x必须是接口类型，案例中列出的每个非接口类型T必须实现x的类型，类型switch的case中列出的类型必须全部不同。

```
TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard "{" { TypeCaseClause } "}" .
TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
TypeCaseClause  = TypeSwitchCase ":" StatementList .
TypeSwitchCase  = "case" TypeList | "default" .
TypeList        = Type { "," Type } .
```

TypeSwitchGuard可以包括一个简短的变量声明。当使用这种形式时，变量被声明在每个子句隐含块的TypeSwitchCase的末尾。在有一个case的子句中，正好列出一个类型，变量具有该类型；否则，变量具有TypeSwitchGuard中表达式的类型。

case可以使用预先声明的标识符nil代替类型；当TypeSwitchGuard中的表达式是一个nil接口值时，就会选择该case。最多可以有一个nil case。

给定一个类型为interface{}的表达式x，下面的类型switch。

```go
switch i := x.(type) {
case nil:
	printString("x is nil")                // type of i is type of x (interface{})
case int:
	printInt(i)                            // type of i is int
case float64:
	printFloat64(i)                        // type of i is float64
case func(int) float64:
	printFunction(i)                       // type of i is func(int) float64
case bool, string:
	printString("type is bool or string")  // type of i is type of x (interface{})
default:
	printString("don't know the type")     // type of i is type of x (interface{})
}
```

可以重写：

```go
v := x  // x is evaluated exactly once
if v == nil {
	i := v                                 // type of i is type of x (interface{})
	printString("x is nil")
} else if i, isInt := v.(int); isInt {
	printInt(i)                            // type of i is int
} else if i, isFloat64 := v.(float64); isFloat64 {
	printFloat64(i)                        // type of i is float64
} else if i, isFunc := v.(func(int) float64); isFunc {
	printFunction(i)                       // type of i is func(int) float64
} else {
	_, isBool := v.(bool)
	_, isString := v.(string)
	if isBool || isString {
		i := v                         // type of i is type of x (interface{})
		printString("type is bool or string")
	} else {
		i := v                         // type of i is type of x (interface{})
		printString("don't know the type")
	}
}
```

在类型switch保护之前可以有一个简单的语句，在保护被评估之前执行。

在类型转换中不允许使用 "fallthrough "语句。

## Switch

https://golang.org/doc/effective_go.html#switch

go的switch比C语言的switch更通用。表达式不需要是常数，甚至不需要是整数，case从上到下进行评估，直到找到匹配的情况，如果switch没有表达式，它就会切换到true。因此，把一个if-else-if-else链写成switch是可能的，也是很习惯的。

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

没有自动跌破，但 case 可以用逗号分隔的列表呈现。

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

虽然它们在Go中并不像其他一些类似C语言那样常见，但break语句可以用来提前终止一个开关。不过有时候，需要脱离周围的循环，而不是switch，在Go中，可以通过给循环加上一个标签，然后对这个标签进行 "break "来实现。这个例子展示了这两种用法。

```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

当然，continue语句也接受一个可选的标签，但它只适用于循环。

在本节的最后，这里有一个 byte slice 的比较例程，它使用了两个switch语句。

```go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

### 类型switch

也可以使用swtich来发现接口变量的动态类型。这样的类型切换使用了类型断言的语法，括号内有关键字type。如果switch在表达式中声明了一个变量，那么该变量在每个子句中都会有相应的类型。在这种情况下重用名称也是一种习惯，实际上是在每个情况下声明一个新的变量，名称相同，但类型不同。

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```