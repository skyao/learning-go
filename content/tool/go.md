# go命名行工具

go是golang安装后自带的命令行工具:

```bash
$ which go
/usr/local/go/bin/go
```

## go build

通过 build 命令执行编译工作：

```bash
go build hello.go
```

在不包含文件名时, go 工具会默认使用当前目录来编译。

```bash
go build
```

可以直接指定包:

```bash
go build github.com/goinaction/code/chapter3/wordcount
```

可以在编译时使用竞争检测器标志来编译程序：

```bash
go build -race
```



## go run

`go run` 命令先编译，然后执行编译创建的可执行程序。

```bash
go run main.go
```

必须指定go文件，不能为空或者给参数"."，这里和 `go build` 命令不同。

编译出来的可执行文件并不会保存，在执行完成后，目录中没有像 `go build` 命令那样生成一个可执行文件。

## go clean

`go clean` 命令执行清理工作。

## go vet

`go vet` 命令会帮开发人员检测代码的常见错误。

```go
func main() {
	fmt.Printf("The quick brown fox jumped over lazy dogs", 3.14)
}
```

这段代码，可以编译通过，执行时也不会报错，但是，因为缺少格式化参数，所以结果会不符合预期：

```bash
The quick brown fox jumped over lazy dogs%!(EXTRA float64=3.14)
```

每次对代码先执行 `go vet` 再将其签入源代码库是一个很好的习惯。

## go fmt

`go fmt` 命令做代码自动格式化并保存，会将代码修改成和 Go 源代码类似的风格。

## go doc

在终端上可以直接使用 go doc 命令来打印文档。

```bash
go doc net
```

无需离开终端,即可快速浏览命令或者包的帮助。

## godoc

在终端会话中输入如下命令：

```bash
godoc -http=:6060
```

在端口 6060 启动 Web 服务器。用浏览器打开 http://localhost:6060 可以看到一个页面，包含所有 Go 标准库和 GOPATH 下的 Go 源代码的文档。