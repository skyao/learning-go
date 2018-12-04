# slice

slice 指向一序列的值，并且包含了长度信息。

`[]T` 是一个元素类型为 `T` 的 slice。

```go
p := []int{2, 3, 5, 7, 11, 13}
```

## 对 slice 切片

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

## 构造 slice

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

## 向 slice 添加元素

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



## 参考资料

- [slice：使用和内幕](http://golang.org/doc/articles/slices_usage_and_internals.html)

