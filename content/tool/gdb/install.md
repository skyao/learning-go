---
date: 2018-12-04T23:50:00+08:00
title: gdb安装
weight: 4021
menu:
  main:
    parent: "tool-gdb"
description : "gdb安装"
---


### 下载

https://www.gnu.org/software/gdb/download/

http://ftp.gnu.org/gnu/gdb

下载最新版本，如 gdb-9.2.tar.gz。

### 从源码构建

参照 README 文档的说明：

```bash
mkdir build
cd build

../configure 
make
make install
```

安装完成之后，验证一下：

```bash
$ which gdb
/usr/local/bin/gdb

$ gdb --version
GNU gdb (GDB) 9.2
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

$ which gcore
/usr/bin/gcore
```

备注：

- 按照上面的方式，只有 gdb/gcore  被安装，没有 gdbserver / libinproctrace.so
- 参考： http://www.linuxfromscratch.org/blfs/view/svn/general/gdb.html

### Mac下codesign的注意事项

在启动台搜索“钥匙串”，打开“钥匙串访问”，“证书助理” -》“创建证书”

- [PermissionsDarwin](https://sourceware.org/gdb/wiki/PermissionsDarwin): 切记按照这个文档的要求进行操作，尤其是 `codesign --entitlements gdb-entitlement.xml -fs gdb-cert $(which gdb)` 这一步
- [MAC上使用gdb(完美解决)](https://blog.csdn.net/github_33873969/article/details/78511733)： 仅做参考，操作时以PermissionsDarwin为准
- [Tips-如何优雅的使用GDB调试Go](https://mp.weixin.qq.com/s/xfDydcpRCmX1dR5FybI0Rw)： 仅做参考，操作时以PermissionsDarwin为准

### Mac下build时需要注意的事项

在mac下可能会遇到如下问题：

```bash
$ go build test.go
$ gdb ./test
GNU gdb (GDB) 9.2
This GDB was configured as "x86_64-apple-darwin19.0.0".
......

Reading symbols from ./test...
(No debugging symbols found in ./test)

```

据说出现这个问题的原因是:

> In Go 1.11, the debug information is compressed for purpose of reduce binary size, and gdb on the Mac does not understand compressed DWARF.
> 
> 在Go 1.11中，为了减少二进制文件的大小，调试信息被压缩了，而Mac上的gdb不理解压缩后的DWARF，所以也可以指定-ldflags=-compressdwarf=false来解决。
>
> The workaround is to also specify -ldflags=-compressdwarf=false which does exactly what it claims.
> 
> 解决办法是指定-ldflags=-compressdwarf=false，这样就可以实现它的要求。

注意在mac 下build golang代码时需要添加 `-ldflags=-compressdwarf=false` 参数：

```bash
$ go build -ldflags=-compressdwarf=false test.go 
$ gdb ./test
GNU gdb (GDB) 9.2
This GDB was configured as "x86_64-apple-darwin19.0.0".
......

Reading symbols from ./test...
Loading Go Runtime support.
```

或者添加环境变量

```bash
export GOFLAGS="-ldflags=-compressdwarf=false"
```

参考资料：

- [在MacOS上使用gdb(cgdb)调试Golang程序](https://www.cnblogs.com/zhuxiaoxi/p/10095097.html)
- [Debug Go program with GDB on macOS](https://stackoverflow.com/questions/52534287/debug-go-program-with-gdb-on-macos)
