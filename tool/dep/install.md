# dep安装

官方参考文档：

https://golang.github.io/dep/docs/installation.html

## Linux下安装

执行下面的安装脚本：

```bash
$> curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh

Warning: the following project(s) have [[constraint]] stanzas in Gopkg.toml:

  ✗  github.com/deckarep/golang-set
  ✗  github.com/julienschmidt/httprouter
  ✗  github.com/magiconair/properties
  ✗  github.com/nu7hatch/gouuid

However, these projects are not direct dependencies of the current project:
they are not imported in any .go files, nor are they in the 'required' list in
Gopkg.toml. Dep only applies [[constraint]] rules to direct dependencies, so
these rules will have no effect.

Either import/require packages from these projects so that they become direct
dependencies, or convert each [[constraint]] to an [[override]] to enforce rules
on these projects, if they happen to be transitive dependencies,

```

由于github直接下载速度不快而且有时被墙，因此最好先翻墙，设置好http_proxy等环境变量再执行命令。



