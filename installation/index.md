# Go安装

## Linux

### Ubuntu Apt-get 安装

在 ubuntu 之上可以用 apt-get 命令简单安装 `golang-1.6`：

```bash
sudo apt-get install golang-1.6
```

> 注： 如果用 apt-get install 命令安装完 golang 之后，再用 apt-get remove 删除，然后再用 apt-get install 命令安装（不要问为什么 ^0^），会发现安装和删除都不干净。此时可以使用 `sudo aptitude remove golang-1.6` 命令完整卸载，再 `sudo aptitude install golang-1.6` 全新安装。

### 手工安装

如果需要安装特定版本，比如最新的 Go 1.7, 就需要手工安装。

先在 go 的官方下载需要的版本，下载地址为：

https://golang.org/dl/

以 Go 1.7 为例， 选择 `go1.7.linux-amd64.tar.gz` 。

然后遵循安装指南的要求，简单解压缩到 `/usr/local` 即可：

https://golang.org/doc/install

```bash
sudo tar -C /usr/local -xzf go1.7.linux-amd64.tar.gz
```

完成之后go就安装在 `/usr/local/go` 下，这也是 Go 默认的安装路径。

### 配置

安装完成之后，需要做两个事情：

1. 将go加入到path中
2. 设置 GOPATH

```bash
# golang
export GOPATH=$HOME/work/soft/go
export PATH=/usr/local/go/bin:$GOPATH/bin:$PATH
```

上面的例子中，将 GOPATH 路径设置为 `$HOME/work/soft/go`，然后将 `/usr/local/go/bin` 和 GOPATH下的bin 加入到 PATH 路径。

> 提醒： 不要用软连接的方式将go连接到 `/usr/bin/go` ，因为后面 golang 在安装一些模块时，会将可执行文件放在 `/usr/lib/go-1.6/bin` 目录下，所以最好将 `/usr/lib/go-1.6/bin` 目录加入 PATH 。

执行 `source /etc/profile` 后执行 `go version` ，查看当前版本：

```bash
$ source /etc/profile
$ go version
go version go1.7 linux/amd64
```

