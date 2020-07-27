---
date: 2020-07-04T23:50:00+08:00
title: Slice类型
weight: 475
menu:
  main:
    parent: "grammar-type"
description : "go语言类型中的slice切片类型"
---

## Slice

> 摘录自 go语言实战

slice 指向一序列的值，并且包含了长度信息。

`[]T` 是一个元素类型为 `T` 的 slice。

```go
p := []int{2, 3, 5, 7, 11, 13}
```

### 对 slice 切片

slice 可以重新切片，创建一个新的 slice 值指向相同的数组。

表达式`s[lo:hi]`表示从 `lo` 到 `hi-1` 的 slice 元素，含两端。因此`s[lo:lo]`是空的，而`s[lo:lo+1]`有一个元素。

```go
p := []int{2, 3, 5, 7, 11, 13}
fmt.Println("p ==", p)
fmt.Println("p[1:4] ==", p[1:4])

// 省略下标代表从 0 开始
fmt.Println("p[:3] ==", p[:3])

// 省略上标代表到 len(s) 结束
fmt.Println("p[4:] ==", p[4:])
```

### 构造 slice

slice 由函数 `make` 创建，第二个参数为数组长度：

```go
a := make([]int, 5)  // len(a)=5，cap(a)=5
```

可以通过第三个参数来指定容量：

```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5
```

注意：slice的长度可以在构造时通过参数明确指定，也可以在切片时通过上下两个下标计算而来。而容量则需要考虑左下标开始位置。

```go
func main() {
	a := make([]int, 5)	// 指定长度为5，容量没有设置，则和长度相同：len=5 cap=5
	printSlice("a", a)
	b := make([]int, 0, 5) // 指定长度为0，容量为5：len=0 cap=5
	printSlice("b", b)
	c := b[:2] // 切片时长度为下表差，容量计算时需要考虑左下标开始位置，这里的左下标从0开始：len=2 cap=5
	printSlice("c", c)
	d := c[2:5] // 切片时长度为下表差，容量计算时需要考虑左下标开始位置，这里的左下标从2开始：len=3 cap=5
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}
```

slice 的零值是 `nil`。一个 nil 的 slice 的长度和容量是 0。

```go
var z []int
fmt.Println(z, len(z), cap(z))
```

### 向 slice 添加元素

 Go 提供了一个内建函数 `append` 向 slice 添加元素：

```go
func append(s []T, vs ...T) []T
```

- `append` 的第一个参数 `s` 是一个类型为 `T` 的数组，其余类型为 `T` 的值将会添加到 slice。
- `append` 的结果是一个包含原 slice 所有元素加上新添加的元素的 slice。
- 如果 `s` 的底层数组太小，而不能容纳所有值时，会分配一个更大的数组。 返回的 slice 会指向这个新分配的数组。

```go
func main() {
	var a []int
	printSlice("a", a)

	// append works on nil slices.
	a = append(a, 0)
	printSlice("a", a)

	// the slice grows as needed.
	a = append(a, 1)
	printSlice("a", a)

	// we can add more than one element at a time.
	a = append(a, 2, 3, 4)
	printSlice("a", a)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}
```

## Slice types

https://golang.org/ref/spec#Slice_types

分片是底层数组的连续段的描述符，它提供了对该数组中元素的编号序列的访问。分片类型表示其元素类型的数组的所有分片的集合。元素的数量称为分片的长度，并且永远不会是负数。未初始化的分片的值为零。

```
SliceType = "[" "]" ElementType .
```

片段s的长度可以通过内置函数len发现；与数组不同，它可能在执行过程中发生变化。元素可以通过0到len(s)-1的整数索引来寻址。一个给定元素的分片索引可能小于底层数组中相同元素的索引。

分片一旦被初始化，总是与持有其元素的底层数组相关联。因此，一个分片与它的数组和同一数组的其他分片共享存储；相反，不同的数组总是代表不同的存储。

分片的底层数组可以延伸到分片的末端。*capacity*/容量是对该范围的衡量：它是切片的长度和切片之外的数组的长度之和；一个长度不大于该容量的切片可以通过从原始切片中切出一个新的切片来创建。切片的*capacity*/容量可以通过内置函数cap(a)来发现。

一个新的、初始化的给定元素类型T的分片值是使用内置函数make制作的，它需要一个分片类型和指定长度和可选容量的参数。用make创建的分片总是分配一个新的、隐藏的数组，返回的分片值指向这个数组。也就是说，执行

```go
make([]T, length, capacity)
```

产生的分片与分配一个数组并对其进行分片一样，所以这两个表达式是等价的：

```go
make([]int, 50, 100)
new([100]int)[0:50]
```

像数组一样，切片总是一维的，但可以组成更高维的对象。对于数组的数组，根据结构，内部数组总是相同的长度；但是对于切片的切片（或切片的数组），内部长度可能会动态变化。此外，内部切片必须单独初始化。

## Slice

https://golang.org/doc/effective_go.html#slices

Slices 包裹了数组，为数据序列提供了一个更通用、更强大、更方便的接口。除了具有显式维度的项目（如变换矩阵），Go 中的大多数数组编程都是通过切片而不是简单的数组来完成的。

切片持有对底层数组的引用，如果您将一个切片分配给另一个切片，则两者都指向同一个数组。如果函数接受切片参数，那么它对切片元素的改变将对调用者可见，类似于传递一个指向底层数组的指针。因此，Read函数可以接受切片参数，而不是一个指针和一个计数；切片中的长度设置了一个读取数据的上限。这里是包os中File类型的Read方法的签名。

```go
func (f *File) Read(buf []byte) (n int, err error)
```

该方法返回读取的字节数和一个错误值（如果有的话）。要读入一个较大的缓冲区buf的前32个字节，需要对缓冲区进行分片（这里用作动词）。

```go
n, err := f.Read(buf[0:32])
```

这样的分片很常见，也很高效。事实上，暂且不说效率，下面的代码段也会读取缓冲区的前32个字节。

```go
    var n int
    var err error
    for i := 0; i < 32; i++ {
        nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
        n += nbytes
        if nbytes == 0 || e != nil {
            err = e
            break
        }
    }
```

分片的长度可以改变，只要它仍然适合于底层数组的限制；只要把它分配给自己的一个分片即可。分片的容量，可以通过内置函数cap访问，报告了分片可能承担的最大长度。这里是一个将数据追加到分片的函数。如果数据超过了容量，分片将被重新分配。返回结果的分片。该函数利用len和cap应用于nil slice时是合法的，并返回0。

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

我们必须在事后返回slice，因为虽然Append可以修改slice的元素，但slice本身（持有指针、长度和容量的运行时数据结构）是通过值传递的。

对 slice 进行追加的想法非常有用，它被 append 内置函数所捕获。不过要理解那个函数的设计，我们需要更多的信息，所以我们稍后会回到它。

### Two-dimensional slices

https://golang.org/doc/effective_go.html#two_dimensional_slices

TODO：后面再细看


### 参考资料

- [slice：使用和内幕](http://golang.org/doc/articles/slices_usage_and_internals.html)： 强烈推荐阅读
- https://gobyexample-cn.github.io/slices
- [Dig101: Go 之灵活的 slice](https://gocn.vip/topics/9586): 推荐阅读

