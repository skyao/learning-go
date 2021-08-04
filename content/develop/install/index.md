---
date: 2018-12-04T23:00:00+08:00
title: Golang安装
weight: 310
description : "介绍Go语言的安装"
---

可以在 Golang 的 [官方网站](https://golang.org/dl/) 下载需要的版本，然后参照 [安装指南](https://golang.org/doc/install) 的说明进行。

## Linux安装

先在 go 的 [官方网站](https://golang.org/dl/) 下载需要的版本， 以 go1.15.6 为例， 选择 `go1.15.6.linux-amd64.tar.gz` 。

然后参照 [安装指南](https://golang.org/doc/install)，简单解压缩到 `/usr/local` 即可：

```bash
sudo tar -C /usr/local -xzf go1.15.6.linux-amd64.tar.gz
```

完成之后go就安装在 `/usr/local/go` 下，这也是 Go 默认的安装路径。

### 配置

安装完成之后，需要将go加入到path中：

```bash
# golang
export PATH=/usr/local/go/bin:$PATH
```

执行 `source /etc/profile` 后执行 `go version` ，查看当前版本：

```bash
$ source /etc/profile
$ go version
go version go1.15.6 linux/amd64
```

执行 `go env`，查看当前 go 的环境变量：

```bash
$ go env
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/root/go"
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
......
```

这里要特别注意的是，golang1.8默认情况下，在 GOPATH 没有设置的情况下，使用的默认路径是 `/root/go` 。这个是非常不好的，因为 `/root` 目录权限非常特殊。

因此最好能自己设置 GOPATH，比如设置到自己的工作区目录，然后将 `GOPATH/bin` 加入到 PATH 中。继续修改 `/etc/profile`：

```bash
export GOPATH=/home/sky/work/soft/gopath
export PATH=/usr/local/go/bin:$GOPATH/bin:$PATH
```

再次执行 `go env`，查看当前 go 的环境变量：

```bash
$ go env
......
GOPATH="/home/sky/work/soft/gopath"
......
```

之后再通过 `go get ****` 命令安装程序时，新的程序就会被安装到 `GOPATH/bin` 下，然后由于`GOPATH/bin` 已经加入到 PATH，因此就可以很方便的使用新安装的程序。

## Windows安装

下载需要的版本，如 `go1.11.windows-amd64.msi `，点击安装。

- 安装过程中选择安装路径为`C:\work\soft\golang`。
- 设置环境变量`GOPATH=C:\work\soft\gopath`
- 设置环境变量`GOROOT=C:\work\soft\golang`
- 修改环境变量PATH，增加内容 `%GOPATH%\bin`

实际安装时发现GOPATH设置无效，`go env`命令看到：

```bash
set GOPATH=C:\Users\aoxia\go
```

## Mac安装

先在 go 的 [官方网站](https://golang.org/dl/) 下载需要的版本，如 `go1.**.darwin-amd64.pkg`，然后双击安装。

更新版本时，同样下载，安装时会提示存在旧版本，然后自动卸载旧版本后安装新版本。

安装路径为 `/usr/local/go/`, 因此也需要修改 profile 文件增加go到path路径：

```bash
# golang
export PATH=/usr/local/go/bin:$PATH
```



或者，用 brew 是最简单的：`brew upgrade golang` 安装， `brew upgrade golang` 升级版本。但实操中发现brew下载golang的速度非常的慢，远不如手工下载安装快。

## 安装后的配置

加速 gomodule：

```bash
export GOPROXY=https://goproxy.cn,direct
export GOSUMDB=sum.golang.google.cn
```

