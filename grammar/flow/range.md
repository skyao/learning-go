# rang遍历

`range` 关键字用来遍历 `list`、`array` 或者 `map`。为了方便理解，可以认为 `range` 等效于 `for earch index of`。

对于 `arrays` 或者 `slices`, 将会返回整型的下标；

```go
// 对于数组，rang返回index
a := [...]string{"a", "b", "c", "d"}
for i := range a {
    fmt.Println("Array item", i, "is", a[i])
}
```

支持返回单值或者两个值， 如果返回一个值，那么为下标，否则为下标和下标所对应的值。

```go
a := [...]string{"a", "b", "c", "d"}
for i, v := range a {
    fmt.Println("Array item", i, "is", v)
}
```

对于 `map`，将会返回下一个键值对的 `key`。 

```go
// 对于map, range 返回 key 
capitals := map[string] string {"France":"Paris", "Italy":"Rome", "Japan":"Tokyo" }
for key := range capitals {
    fmt.Println("Map item: Capital of", key, "is", capitals[key])
}
```

同样支持返回两个值， 直接拿到key和对应的value：

```go
capitals := map[string] string {"France":"Paris", "Italy":"Rome", "Japan":"Tokyo" }
for key, value := range capitals {
    fmt.Println("Map item: Capital of", key, "is", value)
}
```

