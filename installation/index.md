# Go安装

## Linux

### Ubuntu Apt-get 安装

在 ubuntu 之上可以用 apt-get 命令简单安装 `golang-1.6`：

```bash
sudo apt-get install golang-1.6
```

> 注： 如果用 apt-get install 命令安装完 golang 之后，再用 apt-get remove 删除，然后再用 apt-get install 命令安装（不要问为什么 ^0^），会发现安装和删除都不干净。此时可以使用 `sudo aptitude remove golang-1.6` 命令完整卸载，再 `sudo aptitude install golang-1.6` 全新安装。

### 手工安装

如果需要安装特定版本，比如最新的 Go 1.8, 就需要手工安装。

先在 go 的 [官方网站](https://golang.org/dl/) 下载需要的版本， 以 Go 1.8.3 为例， 选择 `go1.8.3.linux-amd64.tar.gz` 。

然后参照 [安装指南](https://golang.org/doc/install)，简单解压缩到 `/usr/local` 即可：

```bash
sudo tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz
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
go version go1.8.3 linux/amd64
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
GCCGO="gccgo"
CC="gcc"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build913026422=/tmp/go-build -gno-record-gcc-switches"
CXX="g++"
CGO_ENABLED="1"
PKG_CONFIG="pkg-config"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
```

这里要特别注意的是，golang1.8默认情况下，在 GOPATH 没有设置的情况下，使用的默认路径是 `/root/go` 。这个是非常不好的，因为 `/root` 目录权限非常特殊。

因此最好能自己设置 GOPATH，比如设置到自己的工作区目录，然后将 `GOPATH/bin` 加入到 PATH 中。继续修改 `/etc/profile`：

```bash
export GOPATH=/home/sky/work/soft/go
export PATH=/usr/local/go/bin:$GOPATH/bin:$PATH
```

之后再通过 `go get ****` 命令安装程序时，新的程序就会被安装到 `GOPATH/bin` 下，然后由于`GOPATH/bin` 已经加入到 PATH，因此就可以很方便的使用新安装的程序。
