---
date: 2020-07-04T23:50:00+08:00
title: 接口定义
weight: 861
menu:
  main:
    parent: "feature-interface"
description : "go语言的接口定义"
---



## go语言实战

### 接口定义和实现

接口类型是由一组方法定义的集合。

```go
// 定义一个interface和它的方法
type Abser interface {
	Abs() float64
}

type Vertex struct {
	X, Y float64
}

// 让结构体实现interface要求的方法
func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

接口类型的值可以存放实现这些方法的任何值。

```go
var a Abser
v := Vertex{3, 4}
a = &v // *Vertex 实现了 Abser
```

### 任意接口类型

在go中，如果要表示类型为任意类型，包括基础类型，可以这样做：

```go
type Value struct {
	v interface{}
}
```

有点类似java中的Object，但是go没有对象继承，也没有Object这种单根继承的root对象，为了表示所有类型，就需要使用interface关键字，而interface在go中是关键字，不是类型，因此要加`{}` 后缀。这个语法相对java有点特别。

## Effective Go

### 接口命名

https://golang.org/doc/effective_go.html#interface-names

按照惯例，单方法接口的命名是由方法名加上 -er 后缀或类似的修饰来构造一个代理名词：Reader, Writer, Formatter, CloseNotifier等等。

这样的名字有很多，尊重它们和它们所使用的函数名是很有成效的。Read、Write、Close、Flush、String等都有规范的标志和含义。为了避免混淆，不要给你的方法起这些名字，除非它有相同的签名和含义。反过来说，如果你的类型实现了一个与一个著名类型上的方法具有相同含义的方法，就给它相同的名称和签名；调用你的字符串转换方法String而不是ToString。

### Generality/通用性

https://golang.org/doc/effective_go.html#generality

如果一个类型只是为了实现一个接口而存在，并且永远不会有超出该接口的导出方法，那么就没有必要导出类型本身。只导出接口，就可以清楚地知道该值除了接口中描述的内容之外没有其他需要注意的行为。这也避免了在一个普通方法的每个实例上重复文档的需要。

在这种情况下，构造函数应该返回一个接口值而不是实现类型。举个例子，在哈希库中，crc32.NewIEEE和adler32.New都返回接口类型hash.Hash32。在Go程序中用CRC-32算法代替Adler-32算法，只需要改变构造函数调用，其余代码不受算法改变的影响。

类似的方法使得各种加密包中的流式密码算法与它们链在一起的块密码算法分离。crypto/cipher包中的Block接口指定了块密码的行为，它提供了单个数据块的加密。那么，通过与bufio包类比，实现这个接口的密码包可以用来构造流密码，用Stream接口表示，而不知道块加密的细节。

加密/密文接口是这样的。

```go
type Block interface {
    BlockSize() int
    Encrypt(dst, src []byte)
    Decrypt(dst, src []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}
```

这里是计数器模式（CTR）流的定义，它将一个块密码变成了一个流密码；注意，块密码的细节被抽象掉了。

```go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```

NewCTR不仅适用于一种特定的加密算法和数据源，而且适用于任何Block接口和任何Stream的实现。因为它们返回的是接口值，所以用其他加密模式替换CTR加密是一个局部的变化。构造函数调用必须被编辑，但由于周围的代码必须只将结果视为Stream，所以它不会注意到这种差异。

### 接口与方法

https://golang.org/doc/effective_go.html#interface_methods

由于几乎任何东西都可以附加方法，所以几乎任何东西都可以满足接口。一个说明性的例子是在http包中，它定义了Handler接口。任何实现Handler的对象都可以服务于HTTP请求。

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

ResponseWriter 本身就是一个接口，它提供了对返回响应给客户端所需方法的访问。这些方法包括标准的 Write 方法，所以 http.ResponseWriter 可以在任何可以使用 io.Writer 的地方使用。Request是一个包含客户端请求的解析表示的结构。

为了简洁起见，我们忽略 POSTs，并假设 HTTP 请求总是 GETs；这种简化并不影响处理程序的设置方式。下面是一个琐碎但完整的处理程序的实现，用来统计页面被访问的次数。

```go
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

(为了配合我们的主题，请注意 Fprintf 如何打印到 http.ResponseWriter。) 作为参考，下面是如何将这样的服务器连接到 URL 树上的一个节点。

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

但为什么要把Counter做成一个结构体呢？一个整数就可以了。接收器需要是一个指针，所以增量对调用者来说是可见的）。

```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

如果你的程序有一些内部状态，需要通知你有一个页面被访问了怎么办？给网页绑定一个频道。

```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

最后，假设我们想在/args上显示调用服务器二进制时使用的参数。很容易写一个函数来打印参数。

```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

我们如何把它变成一个HTTP服务器？我们可以让ArgServer成为某个类型的方法，我们忽略它的值，但是有一个更干净的方法。因为除了指针和接口，我们可以为任何类型定义一个方法，我们可以为一个函数写一个方法。http包中包含了这样的代码。

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

HandlerFunc是一个带方法的类型，ServeHTTP，所以该类型的值可以服务于HTTP请求。看看方法的实现：接收者是一个函数f，方法调用f，这看起来很奇怪，但这和比如说，接收者是一个通道，方法在通道上发送并没有什么不同。

为了使ArgServer成为一个HTTP服务器，我们首先要修改它的签名，使其具有正确的签名。

```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

现在ArgServer与HandlerFunc具有相同的签名，因此可以将其转换为该类型来访问其方法，就像我们将Sequence转换为IntSlice来访问IntSlice.Sort一样。设置它的代码很简洁。

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

当有人访问/args页面时，安装在该页面的处理程序的值为ArgServer，类型为HandlerFunc。HTTP服务器将调用该类型的ServeHTTP方法，ArgServer作为接收方，而接收方又会调用ArgServer（通过HandlerFunc.ServeHTTP内部的调用f(w, req)）。然后，参数将被显示出来。

在这一节中，我们从一个结构、一个整数、一个通道和一个函数制作了一个HTTP服务器，这都是因为接口只是方法的集合，它可以为（几乎）任何类型定义。