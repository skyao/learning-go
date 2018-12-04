# 查看依赖状态

`dep` status命令可以用来查看当前依赖的状态。

也可以配合[graphviz](http://www.graphviz.org/)实现依赖关系的可视化。

参考： https://golang.github.io/dep/docs/daily-dep.html#visualizing-dependencies

## 使用方式

### Linux

```
$ sudo apt-get install graphviz
$ dep status -dot | dot -T png | display
```

### macOS

```
$ brew install graphviz
$ dep status -dot | dot -T png | open -f -a /Applications/Preview.app
```

### Windows

```
> choco install graphviz.portable
> dep status -dot | dot -T png -o status.png; start status.png
```