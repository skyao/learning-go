---
date: 2018-12-04T21:40:00+08:00
title: Go module参考文档
weight: 1751
menu:
  main:
    parent: "organization-module"
description : "Go module参考文档"
---

> 内容翻译自： https://golang.org/ref/mod

## 介紹

模块是Go管理依赖关系的方式。

本文档是Go模块系统的详细参考手册。

## Mudule，package和version

模块是包的集合，这些包被发布，版本控制，并一起分发。模块可以直接从版本控制仓库或模块代理服务器下载。

模块由模块路径（module path）来标识，这个路径在go.mod文件中声明，同时还有模块的依赖信息。模块根目录是包含go.mod文件的目录。主模块是包含go命令被调用的目录的模块。

模块中的每一个包都是同一个目录下的源文件的集合，它们被编译在一起。一个包的路径是与包含包的子目录（相对于模块根目录）连接的模块路径。例如，"golang.org/x/net "模块在 "html "目录下包含一个包。这个包的路径是 "golang.org/x/net/html" 。

### 模块路径

模块路径（module path）是模块的规范名称，在模块的go.mod文件中用 module 指令声明。模块的路径是模块中包路径的前缀。

模块路径应该描述模块的功能和在哪里可以找到它。通常情况下，模块路径包括版本库的根目录，版本库中的目录(通常是空的)，和主要版本的后缀(只适用于version2或更高)。

- 仓库根路径（repository root path）是模块路径中对应于开发模块的版本控制仓库根目录的部分。大多数模块都是在其版本库的根目录下定义的，所以这通常是整个路径。例如，golang.org/x/net 就是同名模块的版本库根目录。

- 如果模块没有定义在版本库的根目录中，模块子目录（module subdirectory ）是模块路径中命名目录的部分，不包括主要版本后缀。这也可以作为语义版本标签的前缀。例如，模块 `golang.org/x/tools/gopls` 在版本库的gopls子目录中，根路径为`golang.org/x/tools`，所以它的模块子目录为gopls。

- 如果模块是在主要版本 2 或更高版本发布的，模块路径必须以主要版本后缀结尾，比如 /v2。这可能是也可能不是子目录名的一部分。例如，路径为 `golang.org/x/repo/sub/v2` 的模块可以放在仓库 `golang.org/x/repo` 的 /sub 或 /sub/v2 子目录下。

如果一个模块可能会被其他模块依赖，那么必须遵循这些规则，这样go命令才能找到并下载这个模块。对于模块路径中允许使用的字符也有一些词法限制。


### 版本

版本（Version）标识了模块的一个不可更改的快照，它可以是一个发布版（release），也可以是一个预发布版（pre-release）。每个版本都以字母v开头，后面是语义版本。有关版本如何格式化、解释和比较的详细信息，请参见 [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)。

概括地说，语义版本由三个非负整数（从左到右分别为主版本/major、次要版本/minor和补丁版本/patch）组成，并由点号分隔。补丁版本的后面可以跟一个以连字符开头的可选的预发布字符串。pre-release 字符串或补丁版本后面可以跟一个以加号开头的构建元数据字符串。例如，v0.0.0、v1.12.134、v8.0.5-pre 和 v2.0.9+meta 是有效的版本。

版本的每个部分都表示该版本是否稳定，是否与以前的版本兼容。

- 主版本（major version）必须递增，而次要版本和补丁版本必须在模块的公共接口或文档功能发生向后不兼容的变化后，例如，在删除一个包后，将其设置为零。
- 在对模块的公共接口或文档功能进行了向后兼容的修改之后，例如，在添加了一个新的功能之后，minor 版本必须递增，补丁版本必须设置为零。
- 补丁版本必须在不影响模块的公共接口的变更之后递增， 例如错误修复或优化。
- pre-release后缀表示一个版本是预发布版本。预发布版本会在相应的发布版本之前进行排序。例如，v1.2.3-pre排在v1.2.3之前。
- 在比较版本时，构建元数据的后缀会被忽略。带有构建元数据的标签在版本控制库中会被忽略，但在 go.mod 文件中指定的版本中会保留构建元数据。后缀 +incompatible 表示在迁移到模块主要版本 2 或更高版本之前发布的版本 。

如果一个版本的主版本是0或者它的后缀是pre-release，那么这个版本就被认为是不稳定的。不稳定的版本不受兼容性要求的限制。例如，v0.2.0 可能与 v0.1.0 不兼容，v1.5.0-beta 可能与 v1.5.0 不兼容。

Go 可能会使用标签、分支或不遵循这些约定的修订版来访问版本控制系统中的模块。然而，在主模块中，go 命令会自动将不遵循此标准的修订名转换为规范版本。作为这个过程的一部分，go 命令也会删除构建元数据的后缀（除了 +incompatible）。这可能会产生一个伪版本，即一个预发布版本，它编码了一个版本标识符（如 Git 提交哈希）和一个来自版本控制系统的时间戳。例如，`go get -d golang.org/x/net@daa7c041` 命令会将提交哈希 daa7c041 转换为伪版本 `v0.0.0-20191109021931-daa7c04131f5`。 在主模块之外，需要使用规范版本，如果在 go.mod 文件中出现像 master 这样的非规范版本，go 命令会报错。

### 伪版本

伪版本(*pseudo-version*)是一种特殊格式的预发布版本，它对版本控制库中的特定版本信息进行编码。例如，`v0.0.0-20191109021931-daa7c04131f5` 就是一个伪版本。

伪版本可能指的是没有语义版本标记的版本。它们可以用来在创建版本标签之前测试提交，例如，在开发分支上。

每个伪版本都有三个部分：

- 基础版本前缀（vX.0.0或vX.Y.Z-0），它来自于修订版之前的语义版本标签，如果没有这样的标签，则是vX.0.0。
- 时间戳 (yymmddhhmmss)，这是修订版创建的 UTC 时间。在 Git 中，这是提交时间，而不是作者时间。
- 版本标识符 (abcdefabcdef)，是提交哈希(commit hash)的 12 个字符前缀，在 Subversion 中，则是一个零填充的版本号。

每一个伪版本可能有三种形式之一，取决于基础版本。这些形式保证了伪版本比它的基础版本高，但比下一个标记的版本低。

- `vX.0.0-yyymmddhhmmss-abcdefabcdef` 是在没有已知基础版本时使用的。与所有版本一样，主要版本X必须与模块的主要版本后缀相匹配。
- `vX.Y.Z-pre.0.yyymmddhhmmss-abcdefabcdef` 在基础版本是像vX.Y.Z-pre这样的预发布版本时使用。
- `vX.Y.(Z+1)-0.yyymmddhhmmss-abcdefabcdef` 在基础版本是像vX.Y.Z这样的发布版本时使用。例如，如果基础版本是 v1.2.3，伪版本可能是 `v1.2.4-0.20191109021931-daa7c04131f5`。

通过使用不同的基础版本，一个以上的伪版本可能会指向同一个提交。当一个较低的版本在伪版本写完后被标记时，这种情况自然会发生。

这些形式赋予伪版本两个有用的属性。

- 具有已知基础版本的伪版本比这些版本排序高，但比其他预发布的后期版本排序低。

- 具有相同基础版本前缀的伪版本按时间顺序排序。

go 命令会执行一些检查，以确保模块作者可以控制伪版本与其他版本的比较，并确保伪版本指的是真正属于模块提交历史的版本:

- 如果指定了基础版本，那么必须有相应的语义版本标签，它是伪版本所描述的版本的祖先。这可以防止开发人员使用比所有标记版本更高的伪版本（如v1.999.999-999999999999-daa7c04131f5）来绕过最小版本选择。

- 时间戳必须与修订版的时间戳相匹配。这可以防止攻击者在模块代理中充斥大量相同的伪版本。这也可以防止模块消费者改变版本的相对顺序。

- 修订版必须是模块仓库的一个分支或标签的祖先。这可以防止攻击者引用未经批准的修改或拉请求。

伪版本从来不需要手工输入。许多命令接受提交哈希或分支名，并会自动将其翻译成伪版本（或标记版本，如果有的话）。例如

```bash
go get -d example.com/mod@master
go list -m -json example.com/mod@abcd1234
```

### 主版本后缀


从主版本2开始，模块路径必须有一个与主要版本相匹配的主版本后缀，比如/v2。例如，如果一个模块的路径 `example.com/mod`是 v1.0.0，那么它的路径 `example.com/mod/v2` 必须是v2.0.0。

主要版本后缀实现了 [导入兼容性规则](https://research.swtch.com/vgo-import)。

> If an old package and a new package have the same import path, the new package must be backwards compatible with the old package.
>
> 如果一个旧的包和一个新的包有相同的导入路径，新的包必须向后兼容旧的包。

根据定义，一个模块的新主版本中的包不能向后兼容之前主版本中的相应包。因此，从v2开始，软件包需要新的导入路径。这可以通过在模块路径中添加主要版本后缀来实现。因为模块路径是模块中每个包的导入路径的前缀，所以在模块路径中添加主要版本后缀可以为每个不兼容的版本提供一个不同的导入路径。

主版本后缀在主版本v0或v1时是不允许的，在v0和v1之间不需要改变模块路径，因为v0版本是不稳定的，没有兼容性保证。此外，对于大多数模块来说，v1与上一个v0版本是向后兼容的；v1版本的作用是对兼容性的承诺，而不是表明与v0相比有不兼容的变化。

作为一种特殊情况，以 `gopkg.in/` 开头的模块路径必须有一个主版本后缀，即使是在 v0 和 v1 时也是如此。 后缀必须以点而不是斜线开头 (例如，gopkg.in/yaml.v2)。

主版本后缀允许一个模块的多个主版本同时存在于同一个编译中。由于 [钻石依赖问题](https://research.swtch.com/vgo-import#dependency_story)，这可能是必要的。通常情况下，如果一个模块在两个不同的版本中被要求使用转义依赖关系，那么将使用较高的版本。然而，如果两个版本不兼容，则两个版本都不能满足所有客户的需求。由于不兼容的版本必须有不同的主要版本号，它们也必须由于主要版本后缀而有不同的模块路径。这就解决了冲突：具有不同后缀的模块被视为独立的模块，它们的包--即使是相对于它们的模块根而言处于同一子目录的包--也是不同的。

许多 Go 项目在迁移到模块之前（也许在模块被引入之前）发布了 v2 或更高版本，但没有使用主版本后缀。这些版本被注解为 +incompatible 的构建标签（例如，v2.0.0+incompatible）。

### 将包解析为模块

当go命令使用包路径加载包时，它需要确定是哪个模块提供了这个包。

go命令首先会在构建列表中搜索路径为包路径前缀的模块。例如，如果导入了包`example.com/a/b`，并且模块 `example.com/a` 在构建列表中，那么go命令将检查`example.com/a`是否在目录b中包含包。在一个目录中必须至少有一个带有.go扩展名的文件才能被认为是一个包。构建约束不用于此目的。如果构建列表中正好有一个模块提供了软件包，那么就使用该模块。如果有两个或更多的模块提供了软件包，则会报错。如果没有模块提供软件包， go 命令将尝试查找一个新的模块 (除非使用 -mod=readonly 或 -mod=vendor 标志，在这种情况下，会报错)。

当go命令为包路径查找新模块时，它会检查 GOPROXY 环境变量，这是一个以逗号分隔的代理URL列表，或者 direct 或 off 的关键字。代理 URL 表示 go 命令应该使用 GOPROXY 协议与模块代理联系。direct 表示 go 命令应该与版本控制系统通信，off 表示不应该尝试通信。GOPRIVATE 和 GONOPROXY 环境变量也可以用来控制这种行为。

对于 GOPROXY 列表中的每个条目，go 命令会请求可能提供软件包的每个模块路径的最新版本（也就是软件包路径的每个前缀）。对于每个成功请求的模块路径，go 命令将以最新的版本下载该模块，并检查该模块是否包含所请求的软件包。如果有一个或多个模块包含所请求的包，则使用路径最长的模块。如果找到一个或多个模块，但没有一个模块包含所要求的包，则报错。如果没有找到模块，go 命令会尝试 GOPROXY 列表中的下一个条目。如果没有剩余的条目，就会报错。

例如，假设 go 命令正在寻找一个提供 `golang.org/x/net/html` 包的模块，并且 GOPROXY 被设置为 `https://corp.example.com,https://proxy.golang.org`。go命令可能会发出以下请求：

- 向 https://corp.example.com/ （并行）：

  - 请求获取最新版本 `golang.org/x/net/html`
  - 请求获取最新版本 `golang.org/x/net`
  - 请求获取最新版本 `golang.org/x`
  - 请求获取最新版本 `golang.org`

- 向 https://proxy.golang.org/  ，如果所有向 https://corp.example.com/ 发起的请求均以404或410失败：

  - 请求获取最新版本 `golang.org/x/net/html`
  - 请求获取最新版本 `golang.org/x/net`
  - 请求获取最新版本 `golang.org/x`
  - 请求获取最新版本 `golang.org`

在找到合适的模块后，go命令会在主模块的 go.mod 文件中添加一个新的 requirement ，包含新模块的路径和版本。这样可以保证以后加载相同的包时，会使用相同版本的模块。如果解析的包没有被主模块中的包导入，新的需求将有一个 `// indirect` 的注释。

## `go.mod` 文件

模块是由其根目录下一个名为 go.mod 的UTF-8编码文本文件定义的。go.mod文件是面向行的。每一行都包含一条指令，由关键字和参数组成。例如

```
module example.com/my/thing

go 1.12

require example.com/other/thing v1.0.2
require example.com/new/thing/v2 v2.3.4
exclude example.com/old/thing v1.2.3
replace example.com/bad/thing v1.4.5 => example.com/good/thing v1.4.5。
```

前面的关键字可以从相邻的行中分解出来，以创建一个块，就像在Go的 import 中一样。

```
require (
    example.com/new/thing/v2 v2.3.4
    example.com/old/thing v1.2.3
)
```

go.mod 文件被设计人可读，机可写。go 命令提供了几个改变 go.mod 文件的子命令。例如，go get 可以升级或降级特定的依赖关系。加载模块图的命令会在需要时自动更新 go.mod。go mod edit 可以进行低级编辑。`golang.org/x/mod/modfile` 包可以被Go程序用来进行同样的程序修改。

### 词汇要素

当一个 go.mod 文件被解析时，它的内容会被分解成一系列的标记。有几种标记：空白/whitespace、注释/comment、标点符号/punctuation、关键字/keyword、标识符/identifier和字符串/string。

空白包括空格(U+0020)、制表符(U+0009)、回车符(U+000D)和换行符(U+000A)。除了换行符之外，其他的白色空格字符没有任何作用，只是用来分隔本来要组合的标记。换行符是重要的标记。

注释从 `//` 开始，一直到行尾。不允许使用 `/* */` 注释。

标点符号包括` (, )`和`=>`。

关键词用来区分go.mod文件中不同类型的指令。允许的关键字有module、go、require、replace和exclude。

标识符是由非空字符组成的序列，例如模块路径(module path)或语义化版本(semantic version)。

字符串是引用的字符序列。有两种字符串：以引号（"，U+0022）开头和结尾的解释字符串和以重音（<，U+0060）开头和结尾的原始字符串。解释后的字符串可能包含由反斜杠(\, U+005C)和其他字符组成的转义序列。转义后的引号(\")不会终止一个解释的字符串。解释字符串的未加引号的值是引号之间的字符序列，每个转义序列由反斜杠后面的字符替换（例如，"/"被"替换，"\n"被n替换）。相反，原始字符串的未引用值只是重音之间的字符序列；反斜杠在原始字符串中没有特殊意义。

在go.mod语法中，标识符和字符串是可以互换的。

### 模块路径和版本

大多数 go.mod 文件中的标识符和字符串都是模块路径或版本。

模块路径必须满足以下要求。

- 路径必须由一个或多个以斜杠 (/, U+002F)分隔的路径元素组成. 它不能以斜线开始或结束。
- 每个路径元素都是由ASCII字母、ASCII数字和有限的ASCII标点符号(`+`, `-`, `.`, `_`, 和 `~`).组成的非空字符串。
- 路径元素不能以点号(., U+002E)开始或结束。
- 元素的前缀到第一个点不能是Windows中的保留文件名，无论大小写 (CON, com1, NuL, 等等)。

如果模块路径出现在require指令中而没有被替换（replace），或者模块路径出现在replace指令的右侧，go命令可能需要下载该路径的模块，必须满足一些额外的要求。

- 前导路径元素（直到第一个斜杠，如果有的话），按照惯例是一个域名，必须只包含小写的ASCII字母、ASCII数字、点（.，U+002E）和破折号（-，U+002D）；它必须至少包含一个点，并且不能以破折号开头。

- 对于/vN形式的最后一个路径元素，其中N看起来是数字(ASCII数字和点)，N不能以前导零开始，不能是/v1，也不能包含任何点。

  - 对于以 `gopkg.in/` 开头的路径， 这个 requirement 被路径遵循 gopkg.in 服务的 requirement 约定替换。

go.mod 文件中的版本可以是规范的或非规范的。

规范版本以字母 v 开头，后面是遵循 Semantic Versioning 2.0.0 规范的语义版本。

大多数其他标识符和字符串都可以用作非规范版本，尽管有一些限制，以避免与文件系统、存储库和模块代理发生问题。非规范版本只允许在主模块的 go.mod 文件中使用。go 命令在自动更新 go.mod 文件时，会尝试用一个等价的规范版本来替换每个非规范版本。

在模块路径与版本相关联的地方 (如 require、replace 和 exclude 指令)，最终的路径元素必须与版本一致。参见主版本后缀。

### 语法

go.mod 语法使用 Extended Backus-Naur Form (EBNF) 指定如下。关于EBNF语法的详细内容请参见go语言规范中的符号部分。

```
GoMod = { Directive } .
Directive = ModuleDirective |
            GoDirective |
            RequireDirective |
            ExcludeDirective |
            ReplaceDirective .
```

Newline、标识符和字符串分别用newline、ident和string表示。

模块路径和版本用 ModulePath 和 Version 表示。

```
ModulePath = ident | string . /* 见上面的限制条件*/
Version = ident | string .    /* 见上面的限制*/
```

### `module` 指令

`module`指令定义了主模块的[路径/Path](https://golang.org/ref/mod#glos-module-path)。`go.mod`文件必须只包含一个 `module` 指令。

```
ModuleDirective = "module" ( ModulePath | "(" newline ModulePath newline ")" newline .
```

例如：

```
module golang.org/x/net
```

### `go` 指令

go指令为模块设置了预期的语言版本，版本必须是有效的Go版本：正整数，后面是点和非负整数（例如，1.9，1.14）。

语言版本决定了在编译模块中的包时，哪些语言功能是可用的。该版本中存在的语言特性将可以使用。在早期版本中删除的语言特性，或者在以后的版本中添加的语言特性将不可用。语言版本不会影响构建标签，它由所使用的 Go 版本决定。

语言版本也用于启用 go 命令中的功能。例如，如果go版本为1.14或更高，就可以启用自动 vendoring 功能。

go.mod 文件最多可以包含一个 go 指令。如果没有go指令，大多数命令都会添加一个当前go版本的go指令。

```
GoDirective = "go" GoVersion newline .
GoVersion = string | ident .  /* valid release version; see above */
```

例如：

```
go 1.14
```

### `require` 指令

`require`指令声明一个给定的模块依赖的最低版本。对于每个所需的模块版本，`go`命令将加载该`go.mod`版本的 文件，并合并该文件中的要求（requirement）。加载所有需求（requirements）后，该`go`命令将使用[最小版本选择（MVS）](https://golang.org/ref/mod#minimal-version-selection)来解析它们，以生成[构建列表](https://golang.org/ref/mod#glos-build-list)。

`go`命令会自动添加`// indirect`一些要求的注释。的`// indirect`评论表明没有从所需的模块包由任何包中的主模块中直接导入。`go`当模块的所选版本高于主模块的其他依赖项已（隐含）暗示的版本时，该命令将添加间接要求。这可能是由于显式升级（`go get -u`），删除了以前施加了要求（`go mod tidy`）的某些其他依赖项，或在其自己的`go.mod`文件中导入没有相应要求的程序包的依赖项（例如`go.mod`完全缺少文件的依赖项） 。

go 命令会自动为一些需求（requirement）添加 `// indirect` 注释。 `// indirect` 注释表明，需求（requirement）模块中的任何包都不会被主模块中的任何包直接导入。当所选模块的版本高于主模块的其他依赖关系所隐含的版本时，go 命令会添加一个间接需求（indirect requirement）。这可能是由于明确的升级 (go get -u)、删除了之前施加需求的其他依赖关系 (go mod tidy)，或者一个依赖关系导入了一个没有在自己的 go.mod 文件中对应需求的包 (例如一个完全没有 go.mod 文件的依赖关系)。

```
RequireDirective = "require" ( RequireSpec | "(" newline { RequireSpec } ")" newline ) .
RequireSpec = ModulePath Version newline .
```

例如：

```
require golang.org/x/net v1.2.3

require (
    golang.org/x/crypto v1.4.5 // indirect
    golang.org/x/text v1.6.7
)
```

### `exclude` 指令

exclude指令可以防止模块的一个版本被go命令加载。如果被排除的版本被 go.mod 文件中的 require 指令引用，那么go命令将列出该模块的可用版本(如go list -m -versions所示)，并加载下一个更高的非排除版本。发布版和预发布版的版本都会被考虑在内，而伪版本则不会。如果没有更高的版本，go 命令将报告一个错误。请注意，这一点在 Go 1.16 中可能会改变。

exclude 指令只适用于主模块的 go.mod 文件，在其他模块中会被忽略。详见最小版本选择。

```
ExcludeDirective = "exclude" ( ExcludeSpec | "(" newline { ExcludeSpec } ")" ) .
ExcludeSpec = ModulePath Version newline .
```

例如：

```
exclude golang.org/x/net v1.2.3

exclude (
    golang.org/x/crypto v1.4.5
    golang.org/x/text v1.6.7
)
```

### `replace` 指令

replace指令可以用其他地方的内容替换模块的特定版本的内容，或者模块的所有版本。替换可以用另一个模块的路径和版本，或者特定平台的文件路径来指定。

如果版本出现在箭头的左侧(=>)，那么只有该模块的特定版本被替换，其他版本将被正常访问。如果左边的版本被省略，则模块的所有版本都会被替换。

如果箭头右边的路径是绝对或相对路径(以./或./开头)，它将被解释为替换模块根目录的本地文件路径，其中必须包含go.mod文件。在这种情况下，必须省略替换的版本。

如果右侧的路径不是本地路径，那么它必须是一个有效的模块路径。在这种情况下，需要一个版本/version。同一模块的版本不能同时出现在构建列表中。

不管是用本地路径还是模块路径指定替换，如果替换的模块有go.mod文件，它的模块指令必须与它替换的模块路径相匹配。

replace指令只适用于主模块的go.mod文件，在其他模块中会被忽略。详见最小版本选择。

```
ReplaceDirective = "replace" ( ReplaceSpec | "(" newline { ReplaceSpec } ")" newline ")" ) .
ReplaceSpec = ModulePath [ Version ] "=>" FilePath newline
            | ModulePath [ Version ] "=>" ModulePath Version newline .
FilePath = /* platform-specific relative or absolute file path */
```

例如：

```
replace golang.org/x/net v1.2.3 => example.com/fork/net v1.4.5

replace (
    golang.org/x/net v1.2.3 => example.com/fork/net v1.4.5
    golang.org/x/net => example.com/fork/net v1.4.5
    golang.org/x/net v1.2.3 => ./fork/net
    golang.org/x/net => ./fork/net
)
```

### 自动更新

在使用模块图时，如果缺少一些信息或者go.mod没有准确地反映现实，go命令会自动更新go.mod。例如，考虑这个go.mod文件：

```
module example.com/M

require (
    example.com/A v1
    example.com/B v1.0.0
    example.com/C v1.0.0
    example.com/D v1.2.3
    example.com/E dev
)

exclude example.com/D v1.2.3
```

更新将非规范的版本标识符改写为规范的semver形式，所以`example.com/A` 的v1变成了v1.0.0，`example.com/E` 的dev变成了dev分支上最新提交的伪版本，可能是`v0.0.0-20180523231146-b3f5c0f6e5f1`。

更新修改了requirement以尊重exclusion，因此排除的 `example.com/D v1.2.3` 的需求更新为使用下一个可用的 `example.com/D` 版本，可能是 v1.2.4 或 v1.3.0。

该更新删除了多余的或误导性的要求。例如，如果`example.com/A v1.0.0` 本身需要 `example.com/B v1.2.0` 和 `example.com/C v1.0.0`，那么go.mod对`example.com/B v1.0.0` 的要求是误导性的（被`example.com/A` 对v1.2.0的需求所取代），而它对`example.com/C v1.0.0`的要求是多余的（被`example.com/A` 对同一版本的需求所暗示），所以两者都将被删除。如果主模块包含了直接从 `example.com/B` 或 `example.com/C` 导入包的包，那么requirement将被保留，但会更新为实际使用的版本。

最后，更新将 go.mod 改成了规范的格式，因此未来的变化将导致最小的差异。如果只需要修改格式，go命令将不会更新go.mod。

因为模块图定义了import语句的含义，任何加载包的命令也会使用，因此会更新go.mod，包括 go build、go get、go install、go list、go test、go mod graph、go mod tidy 和 go mod why。

`-mod=readonly` 标志可以防止命令自动更新 go.mod。然而，如果命令需要执行一个会更新到 go.mod 的操作，它将报错。例如，如果 go build 被要求编译一个没有由 build 列表中任何模块提供的包，go build 会报错，而不是在 go.mod 中查找模块和更新需求。

## 最小版本选择（MVS）

Go使用了一种叫做最小版本选择（Minimal version selection / MVS）的算法来选择一组模块版本，以便在构建包时使用。MVS在Russ Cox的Minimal Version Selection一书中有详细的描述。

从概念上讲，MVS是在用go.mod文件指定的模块的有向图上运行的。图中的每个顶点代表一个模块版本。每条边代表一个依赖关系的最小要求版本，使用require指令指定。主模块的go.mod文件中的替换和排除指令修改了这个图。

MVS 产生的输出是构建列表，即用于构建的模块版本列表。

MVS从主模块（图中一个没有版本的特殊顶点）开始，遍历图，跟踪每个模块的最高要求版本。在遍历结束时，最高要求的版本组成构建列表：它们是满足所有要求的最小版本。

可以使用 `go list -m all` 命令来检查构建列表。与其它依赖关系管理系统不同，构建列表不会保存在一个 "lock" 文件中。MVS是确定性的，当依赖关系的新版本发布时，构建列表不会改变，所以MVS在每个模块感知命令开始时都会计算它。

考虑下图中的例子。主模块需要1.2或更高版本的模块A和1.2或更高版本的模块B。A 1.2和B 1.2分别需要C 1.3和C 1.4。C 1.3和C 1.4都需要D 1.2。

![](images/buildlist.svg)

MVS访问并加载每个模块版本的go.mod文件，用蓝色标出。在图形遍历结束时，MVS 返回一个包含加粗版本的构建列表。A 1.2、B 1.2、C 1.4 和 D 1.2。请注意，B和D的更高版本是可用的，但MVS不选择它们，因为不需要。

### 替换

模块的内容（包括它的go.mod文件）可以通过主模块go.mod文件中的 replace 指令来替换。replace指令可以适用于模块的特定版本，也可以适用于模块的所有版本。

替换会改变模块图，因为被替换的模块可能与被替换的版本有不同的依赖关系。

考虑到下面的例子，C 1.4 被 R 替换了，R 依赖于 D 1.3 而不是 D 1.2，所以 MVS 返回一个包含 A 1.2、B 1.2、C 1.4 (被 R 替换了) 和 D 1.3 的构建列表。

![带有替换的模块版本图](images/replace.svg)

### 排除

在主模块的go.mod文件中使用exclude指令也可以排除模块中的在特定版本。

排除也会改变模块图。当一个版本被排除时，它就会从模块图中被移除，对它的需求会被重定向到下一个更高的版本。

请看下面的例子。C 1.3 已被排除。MVS 将会像 A 1.2 需要 C 1.4（下一个更高版本）而不是 C 1.3 一样。

![带有排除项的模块版本图](images/exclude.svg)

### 升级

go get 命令可用于升级一组模块。要执行升级，go 命令会在运行 MVS 之前通过添加从访问过的版本到升级版本的边来改变模块图。

考虑下面的例子。模块 B 可能从 1.2 升级到 1.3，C 可能从 1.3 升级到 1.4，D 可能从 1.2 升级到 1.3。

![带有升级的模块版本图](https://golang.org/doc/mvs/upgrade.svg)

升级（和降级）可能会增加或删除间接依赖关系。在这种情况下，E 1.1 和 F 1.1 出现在升级后的构建列表中，因为 B 1.3 需要 E 1.1。

为了保留升级，go命令会更新 go.mod 中的requirement。它将把 B 的requirement改为 1.3 版本。它还会添加C 1.4和D 1.3的requirement，并加上间接注释，因为否则不会选择这些版本。

### 降级

go get 命令也可以用来降级一组模块。要执行降级，go 命令会通过删除高于降级版本的版本来改变模块图，同时也会删除依赖于被删除版本的其他模块的版本，因为它们可能与它们所依赖的降级版本不兼容。它也会移除依赖于被移除版本的其他模块的版本，因为这些模块可能与它们所依赖的降级版本不兼容。如果主模块需要一个被降级删除的模块版本，那么requirement就会被改成一个没有被删除的以前的版本。如果没有以前的版本，则requirement会被放弃。

考虑下面的例子。假设发现C 1.4有问题，所以我们降级到C 1.3。C 1.4被从模块图中删除。B 1.2也被删除，因为它需要C 1.4或更高的要求。主模块对B的要求改为1.1。

![降级的模块版本图](https://golang.org/doc/mvs/downgrade.svg)

go get 也可以通过在参数后使用 @none 后缀来完全删除依赖关系。这和降级的工作原理类似。所有被命名的模块的版本都会从模块图中删除。

## 与非模块存储库的兼容性

为了保证从GOPATH到模块的平稳过渡，go命令可以通过添加go.mod文件，从没有迁移到模块的版本库中以模块感知模式下载和构建包。

当 go 命令直接从版本库中下载给定版本的模块时，它会查找模块路径的版本库 URL，将版本映射到版本库中的修订版，然后提取版本库的存档。如果模块的路径等于版本库根目录，而版本库根目录下不包含go.mod文件，go命令就会在模块缓存中合成一个go.mod文件，该文件包含一个模块指令，而没有其他内容。由于合成的 go.mod 文件不包含对其依赖的 require 指令，因此依赖它们的其他模块可能需要额外的 require 指令（带有 // indirect 注释），以确保每次构建时都能以相同的版本获取每个依赖。

当go命令从代理服务器下载模块时，它将go.mod文件与模块的其他内容分开下载。如果原来的模块没有合成的 go.mod 文件，那么代理会提供一个合成的 go.mod 文件。

### `+incompatible` 版本

在主版本2或更高版本发布的模块必须在其路径上有一个与主要版本相匹配的后缀。例如，如果模块是在v2.0.0发布的，那么它的路径必须有一个/v2的后缀。这允许go命令将一个项目的多个主版本视为不同的模块，即使它们是在同一个版本库中开发的。

主版本后缀的要求是在go命令添加模块支持时引入的，而在此之前，许多版本库已经将主版本2或更高版本标记为版本。为了保持与这些版本库的兼容性，go 命令为没有 go.mod 文件的 major version 2 或更高版本添加了 +incompatible 后缀。+incompatible 表示一个版本与主版本号较低的版本属于同一个模块；因此，go 命令可以自动升级到更高的 +incompatible 版本，即使它可能会破坏构建。

考虑下面的requirement示例：

```
require example.com/m v4.1.2+incompatible
```


版本 `v4.1.2+incompatible` 指的是提供模块 `example.com/m` 的仓库中的语义版本标签`v4.1.2`。该模块必须在版本库根目录下（也就是说，版本库根路径也必须是`example.com/m`），而且不能有go.mod文件。模块可能有版本号较低的主版本号，如`v1.5.2`，go命令可能会从这些版本自动升级到 `v4.1.2+incompatible` 版本。

版本v2.0.0被标记后迁移到模块的版本库通常应该发布一个新的主版本。在上面的例子中，作者应该创建一个路径为 `example.com/m/v5` 的模块，并发布 v5.0.0 版本。作者还应该更新模块中的包的导入，使用 `example.com/m/v5` 而不是 `example.com/m` 的前缀。

请注意，+incompatible 后缀不应该出现在版本库中的标签上；像 v4.1.2+incompatible 这样的标签将被忽略。后缀只出现在 go 命令使用的版本中。

还请注意，+incompatible 后缀可能会出现在伪版本中。例如，v2.0.1-20200722182040-012345abcdef+incompatible可能是一个有效的伪版本。

### 最小的模块兼容性

在主版本2或更高版本发布的模块需要在其模块路径上有一个主版本后缀。该模块可能在其版本库中的主版本子目录中开发，也可能不在该子目录中开发，这对在GOPATH模式下导入模块中的包有影响。

通常在 GOPATH 模式下，一个包会存储在一个与它的版本库根目录相匹配的目录中，该目录与版本库中的目录相连接。例如，在版本库中，一个根路径为 `example.com/repo` 的包，其子目录为sub，那么这个包就会存储在 `$GOPATH/src/example.com/repo/sub` 中，并且会被导入为 `example.com/repo/sub` 。

对于一个带有主版本后缀的模块，人们可能期望在`$GOPATH/src/example.com/repo/v2/sub` 目录下找到包 `example.com/repo/v2/sub`。这就需要在其仓库的v2子目录下开发模块。go 命令支持这一点，但并不要求这样做 (参见 Mapping versions to commits)。

如果模块没有在主版本的子目录中开发，那么它在 GOPATH 中的目录将不包含主版本的后缀，而它的包可以在没有主版本后缀的情况下被导入。在上面的例子中，包会在`$GOPATH/src/example.com/repo/sub` 目录下找到，并以 `example.com/repo/sub` 的形式导入。

这就给在模块模式和GOPATH模式下构建的软件包带来了一个问题：模块模式需要后缀，而GOPATH模式则不需要。

为了解决这个问题，在 Go 1.11 中增加了最小模块兼容性，并回传到 Go 1.9.7 和 1.10.3。在GOPATH模式下，当导入路径被解析到一个目录时：

- 当解析 `$modpath/$vn/$dir` 样式的 import 时，其中：

  - `$modpath` 是有效的模块路径，
  - `$vn` 是主要版本的后缀，
  - `$dir` 是一个可能为空的子目录，

- 如果满足以下所有条件：

  - 该包 `$modpath/$vn/$dir` 不在任何相关 [`vendor` 目录中](https://golang.org/ref/mod#glos-vendor-directory)。
  - `go.mod`文件是存在于相同的目录中的文件导入或以任何父目录到`$GOPATH/src`根，
  - 没有 `$GOPATH[i]/src/$modpath/$vn/$suffix` 目录（对于任何root `$GOPATH[i]`），
  - 该文件 `$GOPATH[d]/src/$modpath/go.mod` 存在（针对某个root `$GOPATH[d]`），并将模块路径声明为 `$modpath/$vn`，

- 然后将导入`$modpath/$vn/$dir` 解析到目录 `$GOPATH[d]/src/$modpath/$dir`。

该规则允许已迁移到模块的包在GOPATH模式下构建时导入其他已迁移到模块的包，即使没有使用主版本子目录。


## 模块感知命令

大多数go命令可以在模块感知模式或GOPATH模式下运行。在模块感知模式下，go 命令使用 go.mod 文件来查找版本化的依赖关系，并且通常会从模块缓存中加载包，如果缺少模块，则下载它们。在 GOPATH 模式下，go 命令会忽略模块；它在 vendor 目录和 GOPATH 中查找依赖关系。

每当在当前目录或任何父目录中发现go.mod文件时，模块感知模式默认为激活。为了进行更精细的控制，GO111MODULE 环境变量可以设置为三个值之一：开启、关闭或自动。

- 如果 GO111MODULE=off，go 命令会忽略 go.mod 文件，并以 GOPATH 模式运行。
- 如果 GO111MODULE=on，则 go 命令以模块感知模式运行，即使没有 go.mo 文件存在。并非所有命令在没有go.mod文件的情况下都能工作：请参见 [模块外的模块命令](https://golang.org/ref/mod#commands-outside)。
- 如果 GO111MODULE=auto 或未设置，如果当前目录或任何父目录中存在 go.mod 文件，go 命令将以模块感知模式运行（默认行为）。

在模块感知模式下，GOPATH 不再定义构建过程中 import 的含义，但它仍然会存储下载的依赖项（在 GOPATH/pkg/mod 中；参见 [模块缓存](https://golang.org/ref/mod#module-cache)）和安装的命令（在 GOPATH/bin 中，除非设置了 GOBIN）。


### 构建命令

所有加载包信息的命令都是模块识别的。这包括：

- `go build`
- `go fix`
- `go generate`
- `go get`
- `go install`
- `go list`
- `go run`
- `go test`
- `go vet`

当在模块感知模式下运行时，这些命令使用 go.mod 文件来解释命令行中列出的导入路径或写入 Go 源文件中的路径。这些命令接受以下标志，这些标志对所有模块命令是通用的。

- `-mod` 标志控制 go.mod 是否可以自动更新以及是否使用 vendor 目录。

	* `-mod=mod` 告诉 go 命令忽略 vendor 目录，并自动更新 go.mod，例如，当导入的包没有被任何已知模块提供时。

	* `-mod=readonly` 告诉 go 命令忽略 vendor 目录，并在需要更新 go.mod 时报错。

	* `-mod=vendor` 告诉 go 命令使用 vendor 目录。在这种模式下，go 命令不会使用网络或模块缓存。

	* 默认情况下，如果 go.mod 中的 go 版本是 1.14 或更高，并且存在 vendor 目录，那么 go 命令会像使用了 `-mod=vendor` 一样。否则，go命令将像使用了 `-mod=mod` 一样。

- `-modcacherw` 标志指示 go 命令在模块缓存中创建具有读写权限的新目录， 而不是将它们变成只读。当持续使用这个标志时（通常是通过在环境中设置 `GOFLAGS=-modcacherw` 或运行 `go env -w GOFLAGS=-modcacherw`），模块缓存可以在不改变权限的情况下用 `rm -r` 等命令删除。无论是否使用了 `-modcacherw`，都可以使用 `go clean -modcache` 命令来删除模块缓存。

- `-modfile=file.mod` 标志指示 go 命令读取（也可能写入）模块根目录下的 go.mod 文件。这个文件的名字必须以 `.mod` 结尾。为了确定模块的根目录，一个名为 go.mod 的文件必须仍然存在，但它不会被访问。当指定了 `-modfile` 时，也会使用一个备用的 `go.sum` 文件：它的路径是由 `-modfile` 标志衍生出来的，它修剪了 `.mod` 扩展名，并附加了 `.sum`。

### vendoring

当使用模块时，go 命令通常通过将模块从其源下载到模块缓存中，然后从这些下载的副本中加载包来满足依赖性。vendoring 可以用来允许与旧版本的 Go 进行互操作，或者确保构建过程中使用的所有文件都存储在单一的文件树中。

`go mod vendor` 命令在主模块的根目录下构建了一个名为 vendor 的目录，其中包含了构建和测试主模块中的包所需的所有包的副本。仅由主模块之外的包测试导入的包不包括在内。与 `go mod tidy` 和其他模块命令一样，在构建 vendor 目录时，除了 ignore 之外的构建约束不被考虑。

`go mod vendor` 还创建了 `vendor/modules.txt` 文件，其中包含了 vendored 包的列表和它们被复制的模块版本。当 vendoring 启用时，这个清单被用作模块版本信息的来源，如 `go list -m` 和 `go version -m` 所报告。当go命令读取`vendor/modules.txt` 时，它会检查模块版本是否与go.mod一致。如果在 `vendor/modules.txt` 生成后 `go.mod` 发生了变化，那么 go 命令会报告一个错误，应该再次运行 `go mod vendor` 来更新 vendor 目录。

如果 vendor 目录存在于主模块的根目录中，那么如果主模块的go.mod文件中的go版本是1.14或更高，则会自动使用该目录。要显式地启用vendoring，请使用 `-mod=vendor` 标志调用go命令。要禁用 vendoring，使用标志 `-mod=mod`。

当启用 vendoring 时，像 go build 和 go test 这样的编译命令会从供应商目录加载包，而不是访问网络或本地模块缓存。`go list -m` 命令只打印 go.mod 中列出的模块信息。当启用 vendoring 时，go mod 命令，如 `go mod download` 和 `go mod tidy` 并没有不同的工作方式，仍然会下载模块和访问模块缓存。

与GOPATH中的vendoring不同，go命令会忽略主模块根目录以外位置的厂商目录。

### `go get`

用法：

```
go get [-d] [-t] [-u] [build flags] [packages]
```

例子：

```
# Install the latest version of a tool.
$ go get golang.org/x/tools/cmd/goimports

# Upgrade a specific module.
$ go get -d golang.org/x/net

# Upgrade modules that provide packages imported by packages in the main module.
$ go get -d -u ./...

# Upgrade or downgrade to a specific version of a module.
$ go get -d golang.org/x/text@v0.3.2

# Update to the commit on the module's master branch.
$ go get -d golang.org/x/text@master

# Remove a dependency on a module and downgrade modules that require it
# to versions that don't require it.
$ go get -d golang.org/x/text@none
```

`go get` 命令更新主模块的 `go.mod` 文件中的模块依赖关系，然后构建并安装命令行中列出的包。

第一步是确定要更新的模块，go get 接受包的列表、包模式和模块路径作为参数。如果指定了一个包参数，go get 就会更新提供该包的模块。如果指定了一个包模式 (例如，all 或一个带有...通配符的路径)，go get 会将这个模式扩展为一组包，然后更新提供这些包的模块。如果一个参数命名了一个模块而没有命名包 (例如，golang.org/x/net 模块的根目录下没有包)，go get 将更新模块，但不会构建包。如果没有指定任何参数，go get 就像指定了 .包一样 (当前目录下的包)；这可以和 -u 标志一起使用，以更新提供了 import 包的模块。

每个参数都可以包含一个版本查询后缀，表示所需的版本，如 `go get golang.org/x/text@v0.3.0`。版本查询后缀由 @ 符号和版本查询组成，它可以表示一个特定的版本 (v0.3.0)、一个版本前缀 (v0.3)、一个分支或标签名 (master)、一个修订版本 (1234abcd)，或者一个特殊的查询 latest、upgrade、patch 或 none。如果没有给定版本，go get 使用 @upgrade 查询。

一旦 go get 将其参数解析为特定的模块和版本，go get将在主模块的go.mod文件中添加、更改或删除require指令，以确保模块在未来保持在所需的版本。请注意，go.mod文件中的 require 版本是最小版本，可能会随着新的依赖关系的增加而自动增加。关于如何选择版本以及如何通过模块感知命令解决冲突，请参见 [最小版本选择（MVS）](https://golang.org/ref/mod#minimal-version-selection)。

如果命名模块的新版本需要更高版本的其他模块，则在添加，升级或降级在命令行上命名的模块时，可能会升级其他模块。例如，假设模块`example.com/a`已升级到version `v1.5.0`，而该版本需要module`example.com/b` 为version `v1.2.0`。如果`example.com/b`版本当前需要 模块`v1.1.0`，`go get example.com/a@v1.5.0`还将升级`example.com/b`到 `v1.2.0`。

当命令行上命名的模块增加、升级或降级时，如果命名的模块的新版本需要其他模块的更高的版本，其他模块可能会被升级。例如，假设模块 `example.com/a` 升级到了v1.5.0版本，而该版本需要 v1.2.0 版本的模块 `example.com/b`。如果当前需要模块 `example.com/b` 的 v1.1.0 版本， `go get example.com/a@v1.5.0` 也会将 `example.com/b` 升级到v1.2.0。

![去升级一个传递需求](https://golang.org/doc/mvs/get-upgrade.svg)

当命令行上命名的模块被降级或删除时，其他模块可能会被降级。继续上面的例子，假设模块 `example.com/b` 被降级到v1.1.0。模块 `example.com/a` 也将被降级到需要`example.com/b` 的v1.1.0或更低的版本。

![去降级传递需求](https://golang.org/doc/mvs/get-downgrade.svg)

模块的 requirement 可以使用版本后缀 `@none` 来删除。这是一种特殊的降级。依赖于被移除模块的模块将根据需要被降级或移除。模块的 requirement 可以被移除，即使它的一个或多个包是由主模块的包导入的。在这种情况下，下一个构建命令可能会添加一个新的模块requirement。

如果一个模块在两个不同的版本中被需要（在命令行参数中明确指定或满足升级和降级），go get将报错。

在 go get 更新 go.mod 文件后，它会构建命令行上命名的包。可执行文件将被安装在由GOBIN环境变量命名的目录中，默认为$GOPATH/bin，如果没有设置GOPATH环境变量，则默认为$HOME/go/bin。

`go get` 支持以下标志：

- `-d`标志指示`go get`不要构建或安装软件包。当`-d`使用时，`go get`将只管理依赖关系`go.mod`。
- `-u`标志告诉`go get`升级模块，以提供通过命令行命名的软件包直接或间接导入的软件包。`-u`除非更高版本（预发行版）已要求，否则由选择的每个模块都将升级到最新版本。
- `-u=patch`标志（不是`-u patch`）也告诉`go get`升级的依赖，但`go get`将升级每个依赖于最新的补丁版本（类似`@patch`版本查询）。
- 该`-t`标志`go get`指示考虑在命令行上命名的软件包的构建测试所需要的模块。当`-t`和`-u`一起使用时， `go get`也会更新测试依赖项。
- 该`-insecure`标志不应再使用。它允许`go get`解析自定义导入路径，并使用不安全的方案（例如HTTP）从存储库和模块代理中获取。在`GOINSECURE` [环境变量](https://golang.org/ref/mod#environment-variables)提供了更精细的控制，并应被代替使用。

### `go list -m`

用法：

```
go list -m [-u] [-versions] [list flags] [modules]
```

例如：

```
$ go list -m all
$ go list -m -versions example.com/m
$ go list -m -json example.com/m@latest
```

`-m`标志导致`go list`列出模块而不是包。在这种模式下，to的参数`go list`可以是模块，模块模式（包含 `...`通配符），[版本查询](https://golang.org/ref/mod#version-queries)或特殊模式 `all`，它们与[构建列表](https://golang.org/ref/mod#glos-build-list)中的所有模块匹配。如果未指定任何参数，则列出[主模块](https://golang.org/ref/mod#glos-main-module)。

`-m` 标志会使 go list 列出模块而不是包。在这种模式下，go list 的参数可以是模块、模块模式 (包含...通配符)、版本查询，或者特殊模式 all，即匹配构建列表中的所有模块。如果没有指定参数，则会列出主模块。

列出模块时，该`-f`标志仍然指定应用于Go结构的格式模板，但现在指定为`Module`结构：

当列出模块时，`-f` 标志仍然指定应用于 Go 结构体的格式模板，但现在是 Module 结构体：

```go
type Module struct {
    Path      string       // module path
    Version   string       // module version
    Versions  []string     // available module versions (with -versions)
    Replace   *Module      // replaced by this module
    Time      *time.Time   // time version was created
    Update    *Module      // available update, if any (with -u)
    Main      bool         // is this the main module?
    Indirect  bool         // is this module only an indirect dependency of main module?
    Dir       string       // directory holding files for this module, if any
    GoMod     string       // path to go.mod file for this module, if any
    GoVersion string       // go version used in module
    Error     *ModuleError // error loading module
}

type ModuleError struct {
    Err string // the error itself
}
```

默认输出是打印模块路径，然后是关于版本和替换的信息。例如， go list -m all 可能会被打印出来。

```
example.com/main/module
golang.org/x/text v0.3.0 => /tmp/text
rsc.io/pdf v0.1.1
```

该`Module`结构具有`String`格式化该行输出的方法，因此默认格式等效于`-f '{{.String}}'`。

请注意，`Replace`更换模块后，其字段描述更换模块模块，并且其`Dir`字段设置为更换模块的源代码（如果有）。（也就是说，如果`Replace`为非nil，则将`Dir` 其设置为`Replace.Dir`，而无法访问已替换的源代码。）

该`-u`标志添加有关可用升级的信息。当给定模块的最新版本比当前模块的最新版本时，`list -u`请将模块的 `Update`字段设置为有关新模块的信息。该模块的`String` 方法通过在当前版本之后的方括号中格式化较新的版本来指示可用的升级。例如，`go list -m -u all`可能打印：

```
example.com/main/module
golang.org/x/text v0.3.0 [v0.4.0] => /tmp/text
rsc.io/pdf v0.1.1 [v0.1.2]
```

Module结构有一个String方法来格式化这一行的输出，所以默认格式相当于 `-f '{{.String}}'`。

注意，当模块被替换后，其Replace字段描述替换模块的模块，其Dir字段设置为替换模块的源代码（如果存在的话）。(也就是说，如果Replace是非nil，那么Dir会被设置为Replace.Dir，无法访问被替换的源代码。)

-u标志增加了可用的升级信息。当给定模块的最新版本比当前版本更新时，list -u会将模块的Update字段设置为较新模块的信息。模块的String方法通过在当前版本后的括号内格式化较新的版本来表示可用的升级。例如，go list -m -u all可能会被打印出来。

```
example.com/main/module
golang.org/x/text v0.3.0 [v0.4.0] => /tmp/text
rsc.io/pdf v0.1.1 [v0.1.2]
```

(对于工具来说，`go list -m -u -json all` 可能更方便解析）。

`-versions` 标志会使 list 将模块的 Versions 字段设置为该模块所有已知版本的列表，并按照语义版本从低到高排序。该标志还改变了默认的输出格式，以显示模块路径，然后是以空格分隔的版本列表。

模板函数模块接收一个单一的字符串参数，该参数必须是一个模块路径或查询，并将指定的模块作为一个 Module 结构返回。如果发生错误，结果将是一个带有非零错误字段的模块结构。


### `go mod download`

用法：

``` bash
go mod download [-json] [-x] [modules]
```

例如：

```bash
$ go mod download
$ go mod download golang.org/x/mod@v0.2.0
```

go mod下载命令将指定的模块下载到模块缓存中。参数可以是模块路径，也可以是选择主模块依赖关系的模块模式，或者是 path@version 格式的版本查询。如果没有参数，下载适用于主模块的所有依赖关系。

在平时执行过程中，go命令会根据需要自动下载模块。go mod download 命令主要用于预填模块缓存或加载数据，以便由模块代理提供服务。

默认情况下，download 不会向标准输出写任何东西。它将进度信息和错误打印到标准的错误列表中。

`-json` 标志会使 download 打印一系列 JSON 对象到标准输出，描述每个下载的模块（或失败），对应于这个 Go 结构。

```
type Module struct {
    Path     string // module path
    Version  string // module version
    Error    string // error loading module
    Info     string // absolute path to cached .info file
    GoMod    string // absolute path to cached .mod file
    Zip      string // absolute path to cached .zip file
    Dir      string // absolute path to cached source root directory
    Sum      string // checksum for path, version (as in go.sum)
    GoModSum string // checksum for go.mod (as in go.sum)
}
```

该`-x`标志导致`download`打印命令`download`执行到标准错误。

### `go mod edit`

用法：

```
go mod edit [editing flags] [-fmt|-print|-json] [go.mod]
```

例如：

```
# Add a replace directive.
$ go mod edit -replace example.com/a@v1.0.0=./a

# Remove a replace directive.
$ go mod edit -dropreplace example.com/a@v1.0.0

# Set the go version, add a requirement, and print the file
# instead of writing it to disk.
$ go mod edit -go=1.14 -require=example.com/m@v1.0.0 -print

# Format the go.mod file.
$ go mod edit -fmt

# Format and print a different .mod file.
$ go mod edit -print tools.mod

# Print a JSON representation of the go.mod file.
$ go mod edit -json
```

该`go mod edit`命令提供了一个命令行界面，用于编辑和格式化`go.mod`文件，主要供工具和脚本使用。`go mod edit` 只读取一个`go.mod`文件；它不查找有关其他模块的信息。默认情况下，`go mod edit`读写`go.mod`主模块的文件，但是可以在编辑标记后指定其他目标文件。

编辑标志指定编辑操作的顺序。

- `-module`标志更改模块的路径（`go.mod`文件的模块行）。
- `-go=version`标志设置所需的Go语言版本。
- `-require=path@version`和`-droprequire=path`标志添加和删除指定的模块路径和版本上的要求。请注意，它将`-require` 覆盖上的所有现有要求`path`。这些标志主要用于了解模块图的工具。用户应首选`go get path@version`或`go get path@none`，`go.mod`以根据需要进行其他调整以满足其他模块施加的约束。请参阅[`go get`](https://golang.org/ref/mod#go-get)。
- 在`-exclude=path@version`和`-dropexclude=path@version`标志增加和放弃对给定的模块路径和版本的排除。请注意，`-exclude=path@version`如果该排除已经存在， 则为无操作。
- 该`-replace=old[@v]=new[@v]`标志添加了给定模块路径和版本对的替换。如果省略`@v`in `old@v`，则添加左侧不带版本的替换，该替换适用于旧模块路径的所有版本。如果省略`@v`in `new@v`，则新路径应该是本地模块根目录，而不是模块路径。请注意，它将`-replace` 覆盖的所有多余替换`old[@v]`，因此省略`@v`将删除特定版本的替换。
- 该`-dropreplace=old[@v]`标志丢弃给定模块路径和版本对的替换。如果`@v`提供，则将删除给定版本的替换。左侧没有版本的现有替代品仍然可以替代该模块。如果`@v`省略，则删除没有版本的替换。

可以重复编辑标志。更改将按给定顺序应用。

`go mod edit` 具有控制其输出的其他标志。

- 该`-fmt`标志将重新格式化`go.mod`文件，而不进行其他更改。使用或重写该`go.mod`文件的任何其他修改也暗示了这种重新格式化。唯一需要此标志的时间是如果未指定其他标志，如中所示`go mod edit -fmt`。
- 该`-print`标志`go.mod`以其文本格式打印最终版本，而不是将其写回到磁盘上。
- 该`-json`标志`go.mod`以JSON格式打印最终版本，而不是将其以文本格式写回磁盘。JSON输出对应于以下Go类型：

```
type Module struct {
        Path string
        Version string
}

type GoMod struct {
        Module  Module
        Go      string
        Require []Require
        Exclude []Module
        Replace []Replace
}

type Require struct {
        Path string
        Version string
        Indirect bool
}

type Replace struct {
        Old Module
        New Module
}
```

请注意，这仅描述`go.mod`文件本身，而不描述其他间接引用的模块。对于可用于构建的全套模块，请使用`go list -m -json all`。请参阅[`go list -m`](https://golang.org/ref/mod#go-list-m)。

例如，一个工具能够获得`go.mod`通过解析的输出文件作为数据结构`go mod edit -json`，然后可以通过调用进行更改 `go mod edit`与`-require`，`-exclude`，等。

工具也可以使用该软件包 [`golang.org/x/mod/modfile`](https://pkg.go.dev/golang.org/x/mod/modfile?tab=doc) 来解析，编辑和格式化`go.mod`文件。

### `go mod init`

用法：

```
go mod init [module-path]
```

例如：

```
go mod init
go mod init example.com/m
```

该`go mod init`命令将初始化并`go.mod`在当前目录中写入一个新文件，实际上是在当前目录中创建一个新模块。该`go.mod`文件必须不存在。

`init`接受一个可选参数，即新模块的[模块路径](https://golang.org/ref/mod#glos-module-path)。有关选择[模块路径](https://golang.org/ref/mod#module-path)的说明，请参见模块路径。如果省略了模块路径参数，`init`将尝试使用`.go`文件，vendoring工具配置文件和当前目录（如果在中`GOPATH`）中的导入注释来推断模块路径。

如果存在 vendoring 工具的配置文件，`init`则将尝试从中导入模块要求。`init`支持以下配置文件。

- `GLOCKFILE` (Glock)
- `Godeps/Godeps.json` (Godeps)
- `Gopkg.lock` (dep)
- `dependencies.tsv` (godeps)
- `glide.lock` (glide)
- `vendor.conf` (trash)
- `vendor.yml` (govend)
- `vendor/manifest` (gvt)
- `vendor/vendor.json` (govendor)

vendoring 工具的配置文件不可能总是被完美保真地翻译。例如，如果同一个版本库中的多个包以不同的版本导入，而版本库只包含一个模块，那么导入的go.mod只能要求一个版本的模块。您可能希望运行 go list -m all 来检查构建列表中的所有版本，并运行 go mod tidy 来添加缺失的需求和放弃未使用的需求。

### `go mod tidy`

用法：

```
go mod tidy [-v]
```

`go mod tidy` 确保 go.mod 文件与模块中的源代码相匹配。它添加了构建当前模块的包和依赖关系所需的任何缺失的模块需求，并删除了没有提供任何相关包的模块的需求。它还会向 go.sum 添加任何缺失的条目并删除不必要的条目。

`-v` 标志会使 `go mod tidy` 将删除的模块信息打印成标准错误。

`go mod tidy` 的工作方式是以递归的方式加载主模块中的所有包和它们导入的所有包，这包括测试导入的包（包括其他模块中的测试）。`go mod tidy` 的工作方式就像启用了所有的构建标签一样，所以它会考虑平台特定的源文件和需要自定义构建标签的文件，即使这些源文件通常不会被构建。有一个例外：ignore 构建标签没有被启用，所以一个带有构建约束 // +build ignore 的文件将不会被考虑。需要注意的是，go mod tidy 不会考虑主模块中位于 testdata 目录中的包，或者名字以 .或 _ 开头的包，除非这些包是由其他包明确导入的。

`go mod tidy`通过将所有软件包递归加载到[主模块中](https://golang.org/ref/mod#glos-main-module)以及它们导入的所有软件包来工作。这包括通过测试导入的包（包括其他模块中的测试）。`go mod tidy`就像所有构建标记都已启用一样，因此它将考虑特定于平台的源文件和需要自定义构建标记的文件，即使这些源文件通常不会构建。有一个例外：`ignore`未启用build标记，因此`// +build ignore`将不考虑具有build约束的文件。请注意，`go mod tidy` 不会考虑在包命名的目录在主模块中`testdata`或者与开头的名称`.`或`_`除非这些包明确被其它软件包进口。

一旦 go mod tidy 加载了这组包，它将确保每个提供一个或多个包的模块在主模块的 go.mod 文件中都有一个 require 指令，或者被另一个需要的模块所需要。go mod tidy 将在每个缺失的模块上添加一个最新版本的要求 (最新版本的定义请参见版本查询)。go mod tidy 将会删除那些没有提供任何包的模块的require指令。

go mod tidy 也可以在 require 指令上添加或删除 `// indirect` 注释。`// indirect` 注释表示一个模块没有提供由主模块中的包导入的包。如果在间接依赖中导入包的模块没有go.mod文件，那么这些要求就会存在。如果间接依赖关系的要求版本高于模块图所暗示的版本，它们也可能存在；这通常发生在运行 go get -u ./.... 这样的命令之后。

### `go mod vendor`

用法：

```
go mod vendor [-v]
```

go mod vendor 命令在主模块的根目录下构造了一个名为 vendor 的目录，这个目录包含了支持主模块中包的构建和测试所需的所有包的副本。那些仅由主模块之外的包测试导入的包并不包含在内。与 go mod tidy 和其他模块命令一样，在构建 vendor 目录时，除了 ignore 之外的构建约束不会被考虑。

当启用了vendoring时，go命令将从 vendor 目录中加载包，而不是从模块源下载模块到模块缓存中，并使用那些下载的副本。更多信息请参见 vendoring。

go mod vendor 还会创建 vendor/modules.txt 文件，该文件包含了一个 vendored 包的列表和它们被复制的模块版本。当启用了 vendoring 时，这个清单会被用作模块版本信息的来源，正如 go list -m 和 go version -m 所报告的那样。当go命令读取vendor/modules.txt时，它会检查模块版本是否与go.mod一致。如果自从 vendor/modules.txt 生成后 go.mod 发生了变化，则应重新运行 go mod vendor。

需要注意的是，go mod vendor 在重新构建之前，如果 vendor 目录存在，则会将其删除。不应该对 vendor 目录下的包进行本地修改。go 命令不会检查 vendor 目录中的包是否被修改，但是可以通过运行 go mod vendor 来验证供应商目录的完整性，并检查是否没有被修改。

-v 标志会使 go mod vendor 将 vendored 模块和软件包的名称打印成标准错误。

### `go mod verify`

用法：

```
go mod verify
```

go mod verify 检查存储在模块缓存中的主模块的依赖关系是否在下载后没有被修改。要执行这个检查， go mod verify 会对每个下载的模块.zip 文件和解压目录进行哈希值处理， 然后将这些哈希值与模块第一次下载时记录的哈希值进行比较。 go mod verify 会检查构建列表中的每个模块（可以用 go list -m all 打印）。

如果所有的模块都没有被修改， go mod verify 会打印 "all modules verified"。否则，它会报告哪些模块被修改过，并以非零状态退出。

请注意，所有的模块感知命令都会验证主模块的 go.sum 文件中的哈希值是否与下载到模块缓存中的模块的哈希值相匹配。如果go.sum文件中缺少一个哈希值 (例如，因为模块是第一次使用)，go命令会使用校验数据库来验证它的哈希值 (除非模块路径被GOPRIVATE或GONOSUMDB匹配)。详见认证模块。

与此相反，go mod verify 会检查模块的 .zip 文件及其解压目录的哈希值是否与第一次下载时模块缓存中记录的哈希值相匹配。这对于检测模块缓存中的文件在模块被下载并验证后的变化是很有用的。go mod verify不会下载缓存中没有的模块内容，也不会使用go.sum文件来验证模块内容。但是，go mod verify 可能会下载 go.mod 文件，以便进行最低限度的版本选择，它将使用 go.sum 来验证模块内容。它将使用 go.sum 来验证这些文件，并且它可能会为丢失的哈希值添加 go.sum 项。

### `go version -m`

用法：

```
go version [-m] [-v] [file ...]
```

例如：

```
# Print Go version used to build go.
$ go version

# Print Go version used to build a specific executable.
$ go version ~/go/bin/gopls

# Print Go version and module versions used to build a specific executable.
$ go version -m ~/go/bin/gopls

# Print Go version and module versions used to build executables in a directory.
$ go version -m ~/go/bin/
```

go version报告用于构建命令行上命名的每个可执行文件的Go版本。

如果在命令行中没有命名文件，go version 会打印自己的版本信息。

如果命名了目录，go version 会游历该目录，寻找识别的 Go 二进制文件并报告它们的版本。默认情况下，go版本不会报告在目录扫描中发现的未识别的文件。而 -v 标志则会让它报告未识别的文件。

如果有的话，-m 标志会让 go version 打印每个可执行文件的嵌入式模块版本信息。对于每一个可执行文件，go version -m都会打印出一个像下面这样的以制表符隔开的表格。

```
$ go version -m ~/go/bin/goimports
/home/jrgopher/go/bin/goimports: go1.14.3
        path    golang.org/x/tools/cmd/goimports
        mod     golang.org/x/tools      v0.0.0-20200518203908-8018eb2c26ba      h1:0Lcy64USfQQL6GAJma8BdHCgeofcchQj+Z7j0SXYAzU=
        dep     golang.org/x/mod        v0.2.0          h1:KU7oHjnv3XNWfa5COkzUifxZmxp1TyI7ImMXqFxLwvQ=
        dep     golang.org/x/xerrors    v0.0.0-20191204190536-9bdfabe68543      h1:E7g+9GITq07hpfrRu66IVDexMakfv52eLZ2CXBWiKr4=
```

该表的格式将来可能会更改。可以从这里获得相同的信息：[`runtime/debug.ReadBuildInfo`](https://pkg.go.dev/runtime/debug?tab=doc#ReadBuildInfo)。

表中每一行的含义由第一列中的单词确定。

- **`path`**：`main`用于生成可执行文件的软件包的路径。
- **`mod`**：包含`main`软件包的模块。这些列分别是模块路径，版本和总和。该[主模块](https://golang.org/ref/mod#glos-main-module)具有的版本`(devel)`并没有总和。
- **`dep`**：提供一个或多个链接到可执行文件的软件包的模块。与相同的格式`mod`。
- **`=>`**：[替换](https://golang.org/ref/mod#go-mod-file-replace)上一行中的模块。如果替换是本地目录，则仅列出目录路径（无版本或总和）。如果替换是模块版本，则列出路径，版本和总数，与`mod`和一样`dep`。替换的模块没有总和。

### `go clean -modcache`

用法：

```
go clean [-modcache]
```

`-modcache`标志将导致[`go clean`](https://golang.org/cmd/go/#hdr-Remove_object_files_and_cached_files)删除整个 [模块缓存](https://golang.org/ref/mod#glos-module-cache)，包括版本化依赖项的未打包源代码。

这通常是删除模块缓存的最佳方法。默认情况下，模块缓存中的大多数文件和目录都是只读的，以防止测试和编辑器在通过[身份验证](https://golang.org/ref/mod#authenticating)后意外更改文件 。不幸的是，这会导致命令 `rm -r`失败，因为如果没有先使其父目录可写就无法删除文件。

该`-modcacherw`标志（被[`go build`](https://golang.org/cmd/go/#hdr-Compile_packages_and_dependencies)其他模块知道的命令接受）并导致模块高速缓存中的新目录可写。要传递`-modcacherw`给所有支持模块的命令，请将其添加到 `GOFLAGS`变量中。`GOFLAGS`可以在环境中或通过设置[`go env -w`](https://golang.org/cmd/go/#hdr-Print_Go_environment_information)。例如，以下命令将其永久设置：

```
go env -w GOFLAGS=-modcacherw
```

`-modcacherw`应谨慎使用；开发人员应注意不要对模块缓存中的文件进行更改。[`go mod verify`](https://golang.org/ref/mod#go-mod-verify) 可用于检查高速缓存中的`go.sum`文件是否与主模块文件中的哈希匹配 。

### 版本查询

几个命令允许您使用*版本查询*指定模块的*版本*，该*版本查询*出现在`@`命令行中模块或软件包路径后面的字符之后。

例子：

```
go get example.com/m@latest
go mod download example.com/m@master
go list -m -json example.com/m@e3702bed2
```

版本查询可能是以下之一：

- 完整指定的语义版本（例如）`v1.2.3`，用于选择特定版本。有关语法，请参见[版本](https://golang.org/ref/mod#versions)。
- 语义版本前缀，例如`v1`或`v1.2`，它选择带有该前缀的最高可用版本。
- 语义版本比较，如`<v1.2.3`或`>=v1.5.6`，它选择最近的可用版本给比较对象（最低版本`>`和`>=`，并为最高版本`<`和`<=`）。
- 基础源存储库的修订标识符，例如提交哈希前缀，修订标记或分支名称。如果修订带有语义版本标记，则此查询将选择该版本。否则，此查询将为基础提交选择一个[伪版本](https://golang.org/ref/mod#glos-pseudo-version)。请注意，不能以这种方式选择名称和其他版本查询匹配的分支和标记。例如，查询`v2`选择以开头的最新版本`v2`，而不选择名为的分支`v2`。
- 字符串`latest`，它选择可用的最高发行版本。如果没有发行版本，请`latest`选择最高的预发行版本。如果没有标记版本，请`latest`在存储库默认分支的顶端选择一个伪版本的提交。
- 字符串`upgrade`，类似于`latest`，如果当前需要的模块版本高于`latest`选择的版本（例如，预发行版），`upgrade`则会选择当前版本。
- 字符串`patch`，用于选择最新的可用版本，其主要和次要版本号与当前所需的版本相同。如果当前不需要版本，`patch`则等效于`latest`。

除查询特定命名版本或修订版外，所有查询均考虑由报告的可用版本`go list -m -versions`（请参阅参考资料[`go list -m`](https://golang.org/ref/mod#go-list-m)）。该列表仅包含标记版本，不包含伪版本。不考虑主模块[文件](https://golang.org/ref/mod#glos-go-mod-file)中的[exclude](https://golang.org/ref/mod#go-mod-file-exclude)指令不允许的模块版本。[`go.mod`](https://golang.org/ref/mod#glos-go-mod-file)

[发行版本](https://golang.org/ref/mod#glos-release-version)优于发行前版本。例如，如果版本`v1.2.2`和`v1.2.3-pre`可用，则 `latest`查询将选择`v1.2.2`，即使`v1.2.3-pre`更高。该 `<v1.2.4`查询也会选择`v1.2.2`，即使`v1.2.3-pre`是接近`v1.2.4`。如果没有可用的发行版或预发行版，则`latest`， `upgrade`和`patch`查询将在存储库默认分支的尖端为提交选择一个伪版本。其他查询将报告错误。

### 模块外部的模块命令

感知模块的Go命令通常在工作目录或父目录中的文件所定义的[主模块](https://golang.org/ref/mod#glos-main-module)的上下文中运行`go.mod`。`go.mod`通过将`GO111MODULE`环境变量设置为，某些命令可以在无文件的模块感知模式下运行 `on`。没有`go.mod`文件时，大多数命令的工作方式有所不同。

有关启用和禁用模块感知模式的信息，请参阅[模块感知命令](https://golang.org/ref/mod#mod-commands)。

| 命令                                                         | 行为                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `go build` `go doc` `go fix` `go fmt` `go generate` `go install` `go list` `go run` `go test` `go vet` | 只能`.go`加载，导入和构建标准库中的软件包以及在命令行上指定为文件的软件包 。无法构建来自其他模块的软件包，因为没有地方记录模块需求并确保确定性的构建。 |
| `go get`                                                     | 软件包和可执行文件可以照常构建和安装。请注意，在没有文件的`go get`情况下运行 时没有主模块`go.mod`，因此不会应用`replace`和 `exclude`指令。 |
| `go list -m`                                                 | 除使用该标志外，大多数参数都需要 显式[版本查询](https://golang.org/ref/mod#version-queries)`-versions`。 |
| `go mod download`                                            | 大多数参数都需要 显式[版本查询](https://golang.org/ref/mod#version-queries)。 |
| `go mod edit`                                                | 需要一个显式的文件参数。                                     |
| `go mod graph` `go mod tidy` `go mod vendor` `go mod verify` `go mod why` | 这些命令需要一个`go.mod`文件，如果不存在，将报告错误。       |

## 模块代理

### `GOPROXY` 协议

甲模块代理是能够响应HTTP服务器`GET`用于下面指定的路径的请求。这些请求没有查询参数，并且不需要特定的标头，因此即使从固定文件系统（包括`file://`URL）提供服务的站点也可以作为模块代理。

成功的HTTP响应必须具有状态码200（确定）。重定向（3xx）。状态码为4xx和5xx的响应被视为错误。错误代码404（未找到）和410（消失）指示所请求的模块或版本在代理上不可用，但可以在其他位置找到。错误响应应该有内容类型`text/plain`与 `charset`任一`utf-8`或`us-ascii`。

该`go`命令可以配置为使用`GOPROXY`环境变量联系代理或源控制服务器，该环境变量接受代理URL的列表。该列表可能包含关键字`direct`或`off`（有关详细信息，请参见[环境变量](https://golang.org/ref/mod#environment-variables)）。列表元素可以用逗号（`,`）或竖线（`|`）分隔，以确定错误的后备行为。URL后面带有逗号时，`go`仅在404（未找到）或410（已消失）响应后，该命令才退回到以后的源。当网址后接管道时，`go`在发生任何错误（包括非HTTP错误，例如超时）后，该命令将退回到以后的源。这种错误处理行为使代理可以充当未知模块的网守。例如，对于不在批准列表中的模块，代理可能会以错误403（禁止）响应（请参阅[为私有模块提供服务的私有代理](https://golang.org/ref/mod#private-module-proxy-private)）。

下表指定了模块代理必须响应的查询。对于每个路径，`$base`是代理URL的路径部分，`$module`是模块路径，并且 `$version`是版本。例如，如果代理URL为 `https://example.com/mod`，并且客户端正在`go.mod`为`golang.org/x/text`版本请求模块的文件`v0.3.2`，则客户端将发送的 `GET`请求`https://example.com/mod/golang.org/x/text/@v/v0.3.2.mod`。

为了避免在不区分大小写的文件系统中提供服务时产生歧义，请使用感叹号替换每个大写字母，再加上相应的小写字母，对`$module`和`$version`元素进行大小写编码。这允许模块`example.com/M`与`example.com/m`既存储在磁盘上，因为前者被编码为`example.com/!m`。

| 路径                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `$base/$module/@v/list`          | 以纯文本形式返回给定模块的已知版本的列表，每行一个。此列表不应包含伪版本。 |
| `$base/$module/@v/$version.info` | 返回有关特定版本模块的JSON格式的元数据。响应必须是与以下Go数据结构相对应的JSON对象：`输入信息结构{    版本字符串//版本字符串    时间time.Time //提交时间 } `该`Version`字段是必填字段，并且必须包含有效的 [规范版本](https://golang.org/ref/mod#glos-canonical-version)（请参见 [Versions](https://golang.org/ref/mod#versions)）。该`$version`请求中的路径并不需要是同一版本，甚至一个有效的版本; 该端点可用于查找分支名称或修订版标识符的版本。但是，如果`$version`是主版本与兼容的规范版本 `$module`，则`Version`成功响应中的字段必须相同。该`Time`字段是可选的。如果存在，它必须是RFC 3339格式的字符串。它指示创建版本的时间。将来可能会添加更多字段，因此保留其他名称。 |
| `$base/$module/@v/$version.mod`  | 返回`go.mod`模块特定版本的文件。如果模块没有`go.mod`所请求版本的文件`module` ，则必须返回仅包含具有所请求模块路径的语句的文件。否则，`go.mod`必须返回未经修改的原始文件。 |
| `$base/$module/@v/$version.zip`  | 返回一个zip文件，其中包含模块特定版本的内容。有关必须如何格式化该zip文件的详细信息，请参见[模块zip文件](https://golang.org/ref/mod#zip-files)。 |
| `$base/$module/@latest`          | 以与相同的格式返回有关最新已知模块版本的JSON格式的元数据 `$base/$module/@v/$version.info`。`go`如果`$base/$module/@v/list`为空或没有列出的合适版本，则最新版本应为命令应使用的模块版本。该端点是可选的，并且不需要模块代理即可实现。 |

解析模块的最新版本时，如果找不到合适的版本 ，该`go`命令将请求 。该命令按顺序优先选择：语义上最高的发行版，语义上最高的预发行版以及按时间顺序排列的最新伪版本。在Go 1.12和更早 的版本中，该命令将伪版本视为预发行版本，但是自Go 1.13开始，此不再适用。`$base/$module/@v/list``$base/$module/@latest``go``go``$base/$module/@v/list`

模块代理必须始终为成功的响应`$base/$module/$version.mod`和`$base/$module/$version.zip` 查询提供相同的内容。此内容 使用[文件](https://golang.org/ref/mod#go.sum-files)和默认情况下的 [校验和数据库进行](https://golang.org/ref/mod#checksum-database)[密码认证](https://golang.org/ref/mod#authenticating)。[`go.sum`](https://golang.org/ref/mod#go.sum-files)

该`go`命令会将其从模块代理下载的大多数内容缓存在中的模块缓存中`$GOPATH/pkg/mod/cache/download`。即使直接从版本控制系统下载，该`go`命令也可以合成explicit `info`， `mod`和`zip`文件，并将它们存储在此目录中，就像直接从代理服务器下载它们一样。缓存的布局与代理URL空间相同，因此通过将设置为，可以`$GOPATH/pkg/mod/cache/download`在（或复制到）URL进行服务（或将其复制到） 。`https://example.com/proxy``GOPROXY``https://example.com/proxy`

### 与代理沟通

该`go`命令可以从[模块代理](https://golang.org/ref/mod#glos-module-proxy)下载模块源代码和元数据。的`GOPROXY` [环境变量](https://golang.org/ref/mod#environment-variables)可被用于配置该代理的 `go`命令可以连接到，以及是否可以与直接通信 [版本控制系统](https://golang.org/ref/mod#vcs)。下载的模块数据保存在[模块缓存中](https://golang.org/ref/mod#glos-module-cache)。该`go`命令仅在需要不需要缓存中的信息时才联系代理。

的[`GOPROXY`协议](https://golang.org/ref/mod#goproxy-protocol)部分描述了可被发送至一个请求`GOPROXY`服务器。但是，了解`go`命令何时发出这些请求也很有帮助。例如，请`go build`按照以下步骤操作：

- 通过读取[ 文件](https://golang.org/ref/mod#glos-go-mod-file)并执行[最小版本选择（MVS）来](https://golang.org/ref/mod#glos-minimal-version-selection)计算[构建列表](https://golang.org/ref/mod#glos-build-list)。[`go.mod`](https://golang.org/ref/mod#glos-go-mod-file)
- 阅读在命令行上命名的软件包以及它们导入的软件包。
- 如果构建列表中的任何模块未提供软件包，请找到提供该软件包的模块。将最新版本的模块要求添加到中`go.mod`，然后重新开始。
- 加载完所有内容后生成软件包。

当`go`命令计算构建列表时，它将`go.mod`为[模块图](https://golang.org/ref/mod#glos-module-graph)中的每个模块加载文件。如果`go.mod`文件不在高速缓存中，该`go`命令将使用`$module/@v/$version.mod`请求（其中`$module`是模块路径和 `$version`版本）从代理下载文件 。可以使用诸如之类的工具测试这些请求 `curl`。例如，下面的命令下载版本`go.mod`为的文件 ：`golang.org/x/mod``v0.2.0`

```
$ curl https://proxy.golang.org/golang.org/x/mod/@v/v0.2.0.mod
module golang.org/x/mod

go 1.12

require (
	golang.org/x/crypto v0.0.0-20191011191535-87dc89f01550
	golang.org/x/tools v0.0.0-20191119224855-298f0cb1881e
	golang.org/x/xerrors v0.0.0-20191011141410-1b5146add898
)
```

为了加载程序包，该`go`命令需要提供程序包的模块的源代码。模块源代码分布在`.zip`文件中，这些文件被提取到模块缓存中。如果模块`.zip`不在高速缓存中，则`go`命令将使用`$module/@v/$version.zip`请求将其下载。

```
$ curl -O https://proxy.golang.org/golang.org/x/mod/@v/v0.2.0.zip
$ unzip -l v0.2.0.zip | head
Archive:  v0.2.0.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
     1479  00-00-1980 00:00   golang.org/x/mod@v0.2.0/LICENSE
     1303  00-00-1980 00:00   golang.org/x/mod@v0.2.0/PATENTS
      559  00-00-1980 00:00   golang.org/x/mod@v0.2.0/README
       21  00-00-1980 00:00   golang.org/x/mod@v0.2.0/codereview.cfg
      214  00-00-1980 00:00   golang.org/x/mod@v0.2.0/go.mod
     1476  00-00-1980 00:00   golang.org/x/mod@v0.2.0/go.sum
     5224  00-00-1980 00:00   golang.org/x/mod@v0.2.0/gosumcheck/main.go
```

请注意，即使文件通常包含在文件中，也`.mod`和`.zip`请求是分开的。该命令可能需要下载 许多不同模块的文件，并且文件远小于文件。此外，如果Go项目没有文件，则代理将提供仅包含[ 指令](https://golang.org/ref/mod#go-mod-file-module)的合成文件。从[版本控制系统](https://golang.org/ref/mod#vcs)下载时，该命令会生成合成文件 。`go.mod``.zip``go``go.mod``.mod``.zip``go.mod``go.mod`[`module`](https://golang.org/ref/mod#go-mod-file-module)`go.mod``go`

如果该`go`命令需要加载构建列表中任何模块未提供的软件包，它将尝试查找提供该软件包的新模块。将 [程序包解析为模块部分](https://golang.org/ref/mod#resolve-pkg-mod)介绍了此过程。总之，该`go`命令请求有关可能包含该软件包的每个模块路径的最新版本的信息。例如，对于包`golang.org/x/net/html`，该`go`命令将尝试找到模块的最新版本`golang.org/x/net/html`，`golang.org/x/net`， `golang.org/x/`，和`golang.org`。仅`golang.org/x/net`实际存在并提供该软件包，因此该`go`命令使用该模块的最新版本。如果提供软件包的模块不止一个，该`go`命令将使用路径最长的模块。

当`go`命令请求模块的最新版本时，它首先发送的请求`$module/@v/list`。如果列表为空或不能使用任何返回的版本，它将发送请求`$module/@latest`。选择版本后，该`go`命令将发送`$module/@v/$version.info`元数据请求。然后`$module/@v/$version.mod`，它可以发送并 `$module/@v/$version.zip`请求加载`go.mod`文件和源代码。

```
$ curl https://proxy.golang.org/golang.org/x/mod/@v/list
v0.1.0
v0.2.0

$ curl https://proxy.golang.org/golang.org/x/mod/@v/v0.2.0.info
{"Version":"v0.2.0","Time":"2020-01-02T17:33:45Z"}
```

下载`.mod`或`.zip`文件后，该`go`命令将计算加密哈希，并检查其是否与主模块`go.sum`文件中的哈希匹配 。如果`go.sum`默认情况下不存在哈希，则该`go` 命令从[校验和数据库](https://golang.org/ref/mod#checksum-database)检索它。如果计算的哈希值不匹配，则该`go`命令将报告安全错误，并且不会在模块缓存中安装文件。该`GOPRIVATE`和`GONOSUMDB` [环境变量](https://golang.org/ref/mod#environment-variables)可以用来禁用请求特定模块的校验数据库。`GOSUMDB`也可以将环境变量设置为`off`完全禁用对校验和数据库的请求。请参阅[验证模块](https://golang.org/ref/mod#authenticating)了解更多信息。请注意，为`.info` 请求返回的版本列表和版本元数据未经身份验证，并且可能会随着时间的变化而变化。

## 版本控制系统

该`go`命令可以直接从版本控制存储库下载模块源代码和元数据。从[代理服务器](https://golang.org/ref/mod#communicating-with-proxies)下载模块 通常更快，但是如果没有代理服务器或代理无法访问模块的存储库，则必须直接连接到存储库（对于私有存储库，通常如此）。支持Git，Subversion，Mercurial，Bazaar和Fossil。必须在目录中安装版本控制工具`PATH`，以便 `go`命令使用它。

要从源存储库而不是代理服务器下载特定模块，请设置`GOPRIVATE`或`GONOPROXY`环境变量。要将`go` 命令配置为直接从源存储库下载所有模块，请设置`GOPROXY` 为`direct`。有关更多信息，请参见[环境变量](https://golang.org/ref/mod#environment-variables)。

### 查找模块路径的存储库

当`go`命令以`direct`模式下载模块时，它首先找到包含该模块的存储库。该`go`命令`GET`使用`?go-get=1`查询字符串将HTTP请求发送到从模块路径派生的URL 。例如，对于模块`golang.org/x/mod`，`go`命令可以发送以下请求：

```
https://golang.org/x/mod?go-get=1 (preferred)
http://golang.org/x/mod?go-get=1  (fallback, only with GOINSECURE)
```

该`go`命令将跟随重定向，但是忽略响应状态代码，因此服务器可能会以404或任何其他错误状态进行响应。可以将 `GOINSECURE`环境变量设置为允许回退，并重定向到特定模块的未加密HTTP。

服务器必须以HTML文档作为响应，该HTML文档的文档中包含`<meta>`标记`<head>`。该`<meta>`标签应在文件中出现较早，以避免混淆的`go`命令的限制解析器。特别是，它应该出现在任何原始JavaScript或CSS之前。该`<meta>`标签必须有以下形式：

```
<meta name="go-import" content="root-path vcs repo-url">
```

`root-path`是存储库根路径，是模块路径中与存储库根目录相对应的部分。它必须是请求的模块路径的前缀或完全匹配。如果不完全匹配，则会再次请求前缀以验证`<meta>`标签是否匹配。

`vcs`是版本控制系统。它必须是一个`bzr`，`fossil`，`git`， `hg`，`svn`，`mod`。该`mod`方案指示`go`命令使用[`GOPROXY` 协议](https://golang.org/ref/mod#goproxy-protocol)从给定的URL下载模块。这使开发人员可以在不暴露源存储库的情况下分发模块。

`repo-url`是存储库的URL。如果URL不包括方案，则该 `go`命令将尝试版本控制系统支持的每个协议。例如，使用Git，该`go`命令将`https://`随后尝试`git+ssh://`。仅当模块路径与`GOINSECURE`环境变量匹配时，才可以使用不安全的协议 。

作为示例，请`golang.org/x/mod`再次考虑。该`go`命令向发送请求`https://golang.org/x/mod?go-get=1`。服务器以包含标签的HTML文档作为响应：

```
<meta name="go-import" content="golang.org/x/mod git https://go.googlesource.com/mod">
```

根据该响应，该`go`命令将使用远程URL上的Git存储库`https://go.googlesource.com/mod`。

GitHub和其他流行的托管服务会响应`?go-get=1`所有存储库的查询，因此通常不需要为这些站点上托管的模块配置服务器。

找到存储库URL后，该`go`命令会将存储库克隆到模块高速缓存中。通常，该`go`命令尝试避免从存储库中获取不需要的数据。但是，实际使用的命令因版本控制系统而异，并且可能会随时间而变化。对于Git，该`go`命令可以列出大多数可用版本，而无需下载提交。通常，它将在不下载祖先提交的情况下获取提交，但是有时这样做是必要的。

### 将版本映射到提交

该`go`命令可在特定检查出的存储库中的模块 [规范版本](https://golang.org/ref/mod#glos-canonical-version)等`v1.2.3`，`v2.4.0-beta`或 `v3.0.0+incompatible`。每个模块版本应在存储库中具有语义版本标记，该标记指示应为给定版本检出哪个修订。

如果在存储库根目录或根目录的主版本子目录中定义了模块，则每个版本标记名称都等于相应的版本。例如，模块`golang.org/x/text`是在其存储库的根目录中定义的，因此版本在该存储库中`v0.3.2`具有标签 `v0.3.2`。大多数模块都是如此。

如果在存储库的子目录中定义了[模块](https://golang.org/ref/mod#glos-module-subdirectory)，即模块路径的 [模块子目录](https://golang.org/ref/mod#glos-module-subdirectory)部分不为空，则每个标记名称必须以模块子目录为前缀，后跟斜杠。例如，模块在存储库`golang.org/x/tools/gopls`的`gopls`子目录中以root path定义 `golang.org/x/tools`。该`v0.4.0`模块的版本必须具有`gopls/v0.4.0`在该存储库中命名的标签。

语义版本标记的主版本号必须与模块路径的主版本后缀（如果有）一致。例如，标签`v1.0.0`可能属于模块，`example.com/mod`但不属于模块`example.com/mod/v2`，模块具有像这样的标签`v2.0.0`。

`v2`如果没有`go.mod`文件，并且主版本号或更高版本的标签位于存储库根目录中，则该标签可能属于没有主版本后缀的模块。这种版本以后缀表示 `+incompatible`。版本标记本身不能带有后缀。请参阅 [与非模块存储库的兼容性](https://golang.org/ref/mod#non-module-compat)。

创建标签后，不应将其删除或更改为其他版本。版本经过[认证，](https://golang.org/ref/mod#authenticating)以确保安全，可重复的构建。如果修改了标签，则客户端在下载标签时可能会看到安全错误。即使在删除标签之后，其内容仍可在[模块代理](https://golang.org/ref/mod#glos-module-proxy)上保持可用。

### 将伪版本映射到提交

该`go`命令可以检出特定修订版本的存储库中的模块，该模块编码为[伪版本，](https://golang.org/ref/mod#glos-pseudo-version)例如 `v1.3.2-0.20191109021931-daa7c04131f5`。

伪版本的最后12个字符（`daa7c04131f5`在上面的示例中）表示存储库中有一个要检出的修订。其含义取决于版本控制系统。对于Git和Mercurial，这是提交哈希的前缀。对于Subversion，这是补零的修订版号。

在检出提交之前，该`go`命令将验证时间戳记（`20191109021931`上方）与提交日期匹配。它还验证基本版本（在上面的示例中为`v1.3.1`之前的版本`v1.3.2`）是否对应于作为提交祖先的语义版本标签。这些检查确保模块作者可以完全控制伪版本与其他发行版本的比较。

有关更多信息，请参见[伪版本](https://golang.org/ref/mod#pseudo-versions)。

### 映射分支并提交到版本

可以使用[版本查询](https://golang.org/ref/mod#version-queries)在特定的分支，标签或修订[版中](https://golang.org/ref/mod#version-queries)检出模块 。

```
go get example.com/mod@master
```

该`go`命令将这些名称转换为可用于[最小版本选择（MVS）的](https://golang.org/ref/mod#minimal-version-selection)[规范版本](https://golang.org/ref/mod#glos-canonical-version)。MVS取决于明确订购版本的能力。分支名称和修订版本不能随时间可靠地进行比较，因为它们取决于存储库结构（可能会更改）。

如果某个修订版本标有一个或多个语义版本标签（如）`v1.2.3`，则将使用最高有效版本的标签。该`go`命令仅考虑可能属于目标模块的语义版本标签；例如，由于主要版本与模块路径的后缀不匹配，因此`v1.5.2`不会考虑使用该标记`example.com/mod/v2`。

如果未使用有效的语义版本标记来标记修订，则该`go`命令将生成一个[伪版本](https://golang.org/ref/mod#glos-pseudo-version)。如果修订版本的祖先带有有效的语义版本标签，则将使用祖先的最高版本作为伪版本基础。请参阅[伪版本](https://golang.org/ref/mod#pseudo-versions)。

### 存储库中的模块目录

在特定版本中检出模块的存储库后，该`go` 命令必须找到包含模块`go.mod`文件的目录（模块的根目录）。

回想一下，[模块路径](https://golang.org/ref/mod#module-path)由三部分组成：存储库根路径（对应于存储库根目录），模块子目录和主版本后缀（仅适用于发布至`v2`更高版本的模块 ）。

对于大多数模块，模块路径等于存储库根目录路径，因此模块的根目录是存储库的根目录。

有时在存储库子目录中定义模块。通常，对于具有多个组件的大型存储库，通常需要对其进行发行和版本控制。可以在子目录中找到这样的模块，该子目录与存储库根路径之后的模块路径部分匹配。例如，假设模块`example.com/monorepo/foo/bar`位于根目录为的存储库中`example.com/monorepo`。它的`go.mod`文件必须在`foo/bar`子目录中。

如果模块以主要版本`v2`或更高版本发布，则其路径必须具有 [主要版本后缀](https://golang.org/ref/mod#major-version-suffixes)。可以在两个子目录之一中定义带有主版本后缀的模块：一个带有后缀，另一个不带后缀。例如，假设上面的模块的新版本与path一起发布`example.com/monorepo/foo/bar/v2`。其`go.mod`文件可以在`foo/bar`或中`foo/bar/v2`。

具有主版本后缀的子目录是主版本子目录。它们可用于在单个分支上开发模块的多个主要版本。当在单独的分支上进行多个主要版本的开发时，这可能是不必要的。但是，主要版本子目录具有重要的属性：在`GOPATH`模式下，程序包导入路径与下的目录完全匹配`GOPATH/src`。该`go`命令在`GOPATH`模式下提供最小的模块兼容性（请参阅[与非模块存储库的兼容性](https://golang.org/ref/mod#non-module-compat)），因此与版本内置的项目兼容并不总是需要主要版本子目录 `GOPATH`。不支持最小模块兼容性的较旧工具可能会出现问题。

该`go`命令找到模块根目录后，它将创建`.zip` 目录内容的文件，然后将`.zip`文件提取到模块高速缓存中。有关文件中可能包含哪些文件的详细信息，请参见[文件路径和大小限制](https://golang.org/ref/mod#zip-path-size-constraints)）`.zip`。在将`.zip`文件内容提取到模块高速缓存之前，将对文件内容进行[身份验证](https://golang.org/ref/mod#authenticating)，其方式与`.zip`从代理下载文件时的方式相同。

## 模块zip文件

模块版本作为`.zip`文件分发。由于该`go`命令会自动创建，下载并从[模块代理](https://golang.org/ref/mod#glos-module-proxy)和版本控制存储库中提取文件，因此几乎不需要直接与这些文件进行交互。但是，了解这些文件以了解跨平台兼容性约束或在实现模块代理时仍然很有用。

该[`go mod download`](https://golang.org/ref/mod#go-mod-download)命令下载一个或多个模块的zip文件，然后将这些文件提取到[模块缓存中](https://golang.org/ref/mod#glos-module-cache)。根据[环境变量](https://golang.org/ref/mod#environment-variables)`GOPROXY`和其他[环境变量](https://golang.org/ref/mod#environment-variables)，`go`命令可以从代理下载zip文件或克隆源代码控制存储库并从中创建zip文件。该`-json`标志可用于在模块高速缓存中查找下载zip文件及其提取的内容的位置。

该[`golang.org/x/mod/zip`](https://pkg.go.dev/golang.org/x/mod/zip?tab=doc) 程序包可用于以编程方式创建，提取或检查zip文件的内容。

### 文件路径和大小限制

模块zip文件的内容有很多限制。这些限制确保可以在广泛的平台上安全，一致地提取zip文件。

- 模块zip文件的大小最多为500 MiB。其文件的总未压缩大小也限制为500 MiB。`go.mod`文件限制为16 MiB。 `LICENSE`文件也限制为16 MiB。这些限制的存在是为了减轻对用户，代理和模块生态系统其他部分的拒绝服务攻击。在模块目录树中包含超过500 MiB文件的存储库应在提交时标记模块版本，这些提交仅包括构建模块软件包所需的文件；视频，模型和其他大型资产通常不需要构建。
- 模块zip文件中的每个文件都必须以前缀`$module@$version/`where开头， 其中where`$module`是模块路径和`$version`版本，例如`golang.org/x/mod@v0.3.0/`。模块路径必须有效，版本必须有效且规范，并且版本必须与模块路径的主版本后缀匹配。有关特定的定义和限制，请参见[模块路径和版本](https://golang.org/ref/mod#go-mod-file-ident)。
- 文件模式，时间戳和其他元数据将被忽略。
- 空目录（路径以斜杠结尾的条目）可以包含在模块zip文件中，但不能提取。该`go`命令在其创建的zip文件中不包含空目录。
- 创建zip文件时，符号链接和其他不规则文件将被忽略，因为它们无法跨操作系统和文件系统移植，并且没有可移植的方式以zip文件格式表示它们。
- 在Unicode大小写折叠下，zip文件中的两个文件都不能具有相等的路径（请参阅参考资料[`strings.EqualFold`](https://pkg.go.dev/strings?tab=doc#EqualFold)）。这样可确保可以在不区分大小写的文件系统上提取zip文件而不会发生冲突。
- 一个`go.mod`文件可能会或可能不会出现在顶级目录（`$module@$version/go.mod`）。如果存在，则必须具有名称`go.mod`（全部小写）。命名`go.mod`的文件不允许在任何其他目录中使用。
- 模块中的文件和目录名称可能由Unicode字母，ASCII数字，ASCII空格字符（U + 0020）和ASCII标点字符组成`!#$%&()+,-.=@[]^_{}~`。请注意，程序包路径可能不包含所有这些字符。查看 [`module.CheckFilePath`](https://pkg.go.dev/golang.org/x/mod/module?tab=doc#CheckFilePath) 和 [`module.CheckImportPath`](https://pkg.go.dev/golang.org/x/mod/module?tab=doc#CheckImportPath) 了解差异。
- 文件或目录名到第一个点不能是Windows上的保留文件名，不区分大小写（`CON`，`com1`，`NuL`，等）。

## 专用模块

Go模块经常在公共互联网上不可用的版本控制服务器和模块代理上开发和分发。该`go` 命令可以从专用源下载和构建模块，尽管通常需要进行一些配置。

下面的环境变量可用于配置对私有模块的访问。有关详细信息，请参见[环境变量](https://golang.org/ref/mod#environment-variables)。另请参阅[隐私，](https://golang.org/ref/mod#private-module-privacy)以获取有关控制发送到公用服务器的信息的信息。

- `GOPROXY`—模块代理URL的列表。该`go`命令将尝试按顺序从每个服务器下载模块。关键字`direct`指示`go`命令从开发它们的版本控制存储库中下载模块，而不是使用代理。
- `GOPRIVATE`—模块路径前缀的全局模式列表，应将其视为私有。作为一个默认值`GONOPROXY`和`GONOSUMDB`。
- `GONOPROXY`—不应从代理下载的模块路径前缀的通用模式列表。该`go`命令将从开发它们的版本控制存储库中下载匹配的模块，而不管是否为`GOPROXY`。
- `GONOSUMDB`—不应使用公共校验和数据库[sum.golang.org](https://sum.golang.org/)检查的模块路径前缀的全局模式列表 。
- `GOINSECURE` —可以通过HTTP和其他不安全协议检索的模块路径前缀的全局模式列表。

这些变量可以在开发环境中（例如，在 `.profile`文件中）设置，也可以用永久设置[`go env -w`](https://golang.org/cmd/go/#hdr-Print_Go_environment_information)。

本节的其余部分描述了用于提供对私有模块代理和版本控制存储库的访问的通用模式。

### 私人代理服务所有模块

服务于所有模块（公共和私有）的中央专用代理服务器为管理员提供了最大的控制权，而对单个开发人员的配置要求却最少。

要将`go`命令配置为使用这样的服务器，请设置以下环境变量，用`https://proxy.corp.example.com`您的代理URL和`corp.example.com`模块前缀替换：

```
GOPROXY=https://proxy.corp.example.com
GONOSUMDB=corp.example.com
```

该`GOPROXY`设置指示`go`命令仅从中下载模块 `https://proxy.corp.example.com`；该`go`命令将不会连接到其他代理或版本控制存储库。

该`GONOSUMDB`设置指示`go`命令不要使用公共校验和数据库来验证路径以开头的模块 `corp.example.com`。

以这种配置运行的代理可能需要对私有版本控制服务器的读取权限。它还需要访问公共互联网才能下载公共模块的新版本。

`GOPROXY`可以通过这种方式使用服务器的几种现有实现。最小的实现方式是从[模块缓存](https://golang.org/ref/mod#glos-module-cache)目录中提供文件，并使用[`go mod download`](https://golang.org/ref/mod#go-mod-download)（具有适当的配置）检索丢失的模块。

### 专用代理服务专用模块

专用代理服务器可以为专用模块提供服务，而不必同时提供公共可用的模块。该`go`命令可以配置退回到公共资源的不可用的专用服务器模块。

要配置`go`命令以这种方式工作，请设置以下环境变量，用`https://proxy.corp.example.com`代理URL和 `corp.example.com`模块前缀替换：

```
GOPROXY=https://proxy.corp.example.com,https://proxy.golang.org,direct
GONOSUMDB=corp.example.com
```

该`GOPROXY`设置指示`go`命令首先尝试下载模块 `https://proxy.corp.example.com`。如果该服务器以404（未找到）或410（已消失）响应，则该`go`命令将退回到 `https://proxy.golang.org`，然后将连接定向到存储库。

该`GONOSUMDB`设置指示`go`命令不要使用公共校验和数据库来验证路径以开头的模块 `corp.example.com`。

请注意，此配置中使用的代理仍然可以控制对公共模块的访问，即使它不为公共模块提供服务。如果代理对错误状态不是404或410的请求作出响应，则该`go`命令将不会退回到`GOPROXY`列表中的后续条目。例如，对于具有不合适许可证或已知安全漏洞的模块，代理可能会以403（禁止）响应。

### 直接访问私有模块

该`go`命令可以配置为绕过公共代理，并直接从版本控制服务器下载私有模块。当运行私有代理服务器不可行时，这很有用。

要配置该`go`命令以这种方式工作，请设置`GOPRIVATE`，替换 `corp.example.com`专用模块前缀：

```
GOPRIVATE=corp.example.com
```

该`GOPROXY`变量不需要在这种情况下进行更改。默认为`https://proxy.golang.org,direct`，指示该`go`命令首先尝试下载模块`https://proxy.golang.org`，然后如果该代理以404（未找到）或410（已消失）响应，则退回到直接连接。

该`GOPRIVATE`设置指示`go`命令不要连接以开头的模块的代理或校验和数据库`corp.example.com`。

可能仍需要内部HTTP服务器来[解析存储库URL的模块路径](https://golang.org/ref/mod#vcs-find)。例如，当`go`命令下载模块时`corp.example.com/mod`，它将向发送GET请求 `https://corp.example.com/mod?go-get=1`，并在响应中查找存储库URL。为避免此要求，请确保每个专用模块路径都有一个VCS后缀（如`.git`），用于标记存储库根前缀。例如，当`go`命令下载模块`corp.example.com/repo.git/mod`，它将在克隆Git仓库`https://corp.example.com/repo.git`或 `ssh://corp.example.com/repo.git`无需进行额外的请求。

开发人员将需要对包含私有模块的存储库的读取权限。这可以在全局VCS配置文件（如）中进行配置`.gitconfig`。最好将VCS工具配置为不需要交互式身份验证提示。默认情况下，调用Git时，该`go`命令通过设置禁用交互式提示`GIT_TERMINAL_PROMPT=0`，但它遵循显式设置。

### 将凭据传递给私人代理

与代理服务器通信时，该`go`命令支持HTTP[基本认证](https://en.wikipedia.org/wiki/Basic_access_authentication)。

凭证可以在[`.netrc` 文件中](https://www.gnu.org/software/inetutils/manual/html_node/The-_002enetrc-file.html)指定。例如，`.netrc`包含下面几行的文件将配置`go` 命令以`proxy.corp.example.com`使用给定的用户名和密码连接到计算机。

```
machine proxy.corp.example.com
login jrgopher
password hunter2
```

可以使用`NETRC`环境变量设置文件的位置。如果 `NETRC`未设置，则该`go`命令将`$HOME/.netrc`在类似UNIX的平台或`%USERPROFILE%\_netrc`Windows上读取。

中的字段`.netrc`用空格，制表符和换行符分隔。不幸的是，这些字符不能在用户名或密码中使用。另请注意，计算机名称不能是完整的URL，因此无法为同一计算机上的不同路径指定不同的用户名和密码。

或者，可以直接在`GOPROXY`URL中指定凭据。例如：

```
GOPROXY=https://jrgopher:hunter2@proxy.corp.example.com
```

采用这种方法时请小心：环境变量可能会出现在Shell历史记录和日志中。

### 隐私

该`go`命令可以从模块代理服务器和版本控制系统下载模块和元数据。环境变量`GOPROXY` 控制使用哪些服务器。环境变量`GOPRIVATE`并 `GONOPROXY`控制从代理获取哪些模块。

的默认值为`GOPROXY`：

```
https://proxy.golang.org,direct
```

使用此设置，当`go`命令下载模块或模块元数据时，它将首先向`proxy.golang.org`Google发送的公共模块代理（[隐私政策](https://proxy.golang.org/privacy)）发送请求。有关每个请求中发送哪些信息的详细信息，请参见 [`GOPROXY`协议](https://golang.org/ref/mod#goproxy-protocol)。该`go`命令不会传输可识别个人身份的信息，但会传输所请求的完整模块路径。如果代理以404（未找到）或410（已消失）状态响应，则该`go`命令将尝试直接连接到提供该模块的版本控制系统。有关详细信息，请参见[版本控制系统](https://golang.org/ref/mod#vcs)。

在`GOPRIVATE`或`GONOPROXY`环境变量可能是私有，不应该由任何代理请求被设置为匹配模块前缀glob模式列表。例如：

```
GOPRIVATE=*.corp.example.com,*.research.example.com
```

`GOPRIVATE`只是充当一个默认的`GONOPROXY`和`GONOSUMDB`，所以它不是必要设置`GONOPROXY`，除非`GONOSUMDB`应该有不同的值。当模块路径与匹配时`GONOPROXY`，该`go`命令将忽略 `GOPROXY`该模块，并直接从其版本控制存储库中获取该模块。当没有代理提供私有模块时，此功能很有用。请参阅[直接访问专用模块](https://golang.org/ref/mod#private-module-proxy-direct)。

如果存在[为所有模块提供服务](https://golang.org/ref/mod#private-module-proxy-all)的[受信任代理](https://golang.org/ref/mod#private-module-proxy-all)，则`GONOPROXY`不应设置。例如，如果`GOPROXY`设置为一个源，则该`go`命令不会从其他源下载模块。 `GONOSUMDB`在这种情况下仍应设置。

```
GOPROXY=https://proxy.corp.example.com
GONOSUMDB=*.corp.example.com,*.research.example.com
```

如果存在[仅服务于私有模块](https://golang.org/ref/mod#private-module-proxy-private)的[受信任代理](https://golang.org/ref/mod#private-module-proxy-private)，则`GONOPROXY`不应设置，但是必须注意确保代理以正确的状态代码响应。例如，考虑以下配置：

```
GOPROXY=https://proxy.corp.example.com,https://proxy.golang.org
GONOSUMDB=*.corp.example.com,*.research.example.com
```

假设由于输入错误，开发人员尝试下载不存在的模块。

```
go mod download corp.example.com/secret-product/typo@latest
```

该`go`命令首先从请求此模块`proxy.corp.example.com`。如果该代理以404（未找到）或410（已消失）响应，则`go`命令将退回到`proxy.golang.org`，`secret-product`以在请求URL中传输路径。如果专用代理以任何其他错误代码响应，该`go` 命令将显示错误，并且不会退回到其他来源。

除了代理之外，该`go`命令还可以连接到校验和数据库，以验证未在中列出的模块的加密哈希`go.sum`。在`GOSUMDB` 环境变量设置名称，URL和校验数据库的公共密钥。默认值`GOSUMDB`是`sum.golang.org`由谷歌（运营的公共校验数据库[的隐私政策](https://sum.golang.org/privacy)）。有关每个请求传输的内容的详细信息，请参见[Checksum数据库](https://golang.org/ref/mod#checksum-database)。与代理一样，该`go`命令不会传输个人身份信息，但会传输所请求的完整模块路径，并且校验和数据库无法计算非公共模块的校验和。

可以将`GONOSUMDB`环境变量设置为指示哪些模块是私有模块的模式，并且不应从校验和数据库中请求该模块。`GOPRIVATE`作为一个默认`GONOSUMDB`和`GONOPROXY`，所以它不是必要设置`GONOSUMDB`，除非`GONOPROXY`应该有不同的值。

代理可以[镜像校验和数据库](https://go.googlesource.com/proposal/+/master/design/25530-sumdb.md#proxying-a-checksum-database)。如果代理`GOPROXY`执行此操作，则该`go`命令将不会直接连接到校验和数据库。

`GOSUMDB`可以设置为`off`完全禁用校验和数据库。使用此设置，该`go`命令将不会对下载的模块进行身份验证，除非它们已经在中`go.sum`。请参阅[验证模块](https://golang.org/ref/mod#authenticating)。

## 模块缓存

该模块的缓存是其中的目录`go`命令店下载模块文件。模块缓存与构建缓存不同，构建缓存包含已编译的程序包和其他构建工件。

模块缓存的默认位置是`$GOPATH/pkg/mod`。要使用其他位置，请设置`GOMODCACHE` [环境变量](https://golang.org/ref/mod#environment-variables)。

模块缓存没有最大大小，该`go`命令不会自动删除其内容。

缓存可以由同一台机器上开发的多个Go项目共享。`go`无论主模块的位置如何，该命令都将使用相同的缓存。该`go`命令的多个实例可以安全地同时访问同一模块高速缓存。

该`go`命令使用只读权限在缓存中创建模块源文件和目录，以防止在模块下载后意外更改。不幸的是，这种副作用使高速缓存很难通过诸如之类的命令来删除`rm -rf`。可以使用删除缓存 [`go clean -modcache`](https://golang.org/ref/mod#go-clean-modcache)。或者，使用该`-modcacherw`标志时 ，该`go`命令将创建具有读写权限的新目录。这增加了编辑器，测试和其他程序修改模块缓存中文件的风险。该[`go mod verify`](https://golang.org/ref/mod#go-mod-verify)命令可用于检测对主模块相关性的修改。它扫描每个模块依赖项的提取内容，并确认它们与中期望的哈希值匹配`go.sum`。

下表说明了模块高速缓存中大多数文件的用途。一些临时文件（锁定文件，临时目录）被省略。对于每个路径， `$module`是模块路径，并且`$version`是版本。以斜杠（`/`）结尾的路径是目录。使用感叹号对模块路径和版本中的大写字母进行转义（`Azure`以进行转义`!azure`），以避免在不区分大小写的文件系统上发生冲突。

| 路径                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| `$module@$version/`                          | 包含提取的模块`.zip` 文件内容的目录。这用作已下载模块的模块根目录。`go.mod`如果原始模块没有一个文件，它将不包含文件。 |
| `cache/download/`                            | 目录，包含从模块代理下载的文件和从[版本控制系统](https://golang.org/ref/mod#vcs)派生的文件。该目录的布局遵循 [`GOPROXY`协议](https://golang.org/ref/mod#goproxy-protocol)，因此当由HTTP文件服务器提供服务或使用`file://`URL引用时，此目录可用作代理。 |
| `cache/download/$module/@v/list`             | 已知版本列表（请参阅 [`GOPROXY`协议](https://golang.org/ref/mod#goproxy-protocol)）。这可能会随着时间而改变，因此该`go`命令通常会获取新副本，而不是重新使用此文件。 |
| `cache/download/$module/@v/$version.info`    | 有关版本的JSON元数据。（请参阅 [`GOPROXY`协议](https://golang.org/ref/mod#goproxy-protocol)）。这可能会随着时间而改变，因此该`go`命令通常会获取新副本，而不是重新使用此文件。 |
| `cache/download/$module/@v/$version.mod`     | `go.mod`此版本 的文件（请参阅 [`GOPROXY`协议](https://golang.org/ref/mod#goproxy-protocol)）。如果原始模块没有`go.mod`文件，则这是没有要求的综合文件。 |
| `cache/download/$module/@v/$version.zip`     | 模块的压缩内容（请参阅 [`GOPROXY`协议](https://golang.org/ref/mod#goproxy-protocol)和 [模块zip文件](https://golang.org/ref/mod#zip-files)）。 |
| `cache/download/$module/@v/$version.ziphash` | 文件中`.zip`文件的加密哈希。请注意，`.zip`文件本身未进行哈希处理，因此文件顺序，压缩，对齐方式和元数据不会影响哈希。使用模块时，`go`命令将验证此哈希值是否与中的相应行匹配 [`go.sum`](https://golang.org/ref/mod#go.sum-files)。该 [`go mod verify`](https://golang.org/ref/mod#go-mod-verify)命令检查模块`.zip`文件和提取的目录的哈希是否与这些文件匹配。 |
| `cache/download/sumdb/`                      | 包含从[校验和数据库](https://golang.org/ref/mod#checksum-database)下载的文件的目录 （通常为 `sum.golang.org`）。 |
| `cache/vcs/`                                 | 包含直接从其来源获取的模块的克隆版本控制存储库。目录名称是从存储库类型和URL派生的十六进制编码的哈希。存储库针对磁盘大小进行了优化。例如，克隆的Git存储库在可能的情况下是裸露的且浅薄的。 |

## 认证模块

当`go`命令将模块[zip文件](https://golang.org/ref/mod#zip-files)或[`go.mod` 文件](https://golang.org/ref/mod#go-mod-files)下载到[模块高速缓存中时](https://golang.org/ref/mod#module-cache)，它会计算一个密码哈希，并将其与一个已知值进行比较，以验证自文件首次下载以来该文件没有更改。`go`如果下载的文件没有正确的哈希，该命令将报告安全错误。

对于`go.mod`文件，命令将`go`根据文件内容计算哈希值。对于模块zip文件，该`go`命令以确定的顺序根据存档中文件的名称和内容计算哈希。哈希不受文件顺序，压缩，对齐和其他元数据的影响。有关[`golang.org/x/mod/sumdb/dirhash`](https://pkg.go.dev/golang.org/x/mod/sumdb/dirhash?tab=doc) 哈希实现的详细信息，请参见 。

该`go`命令将每个哈希与主模块[`go.sum`文件中](https://golang.org/ref/mod#go.sum-files)的相应行进行比较。如果哈希值与中的哈希值不同`go.sum`，该`go`命令将报告一个安全错误，并删除下载的文件而不将其添加到模块缓存中。

如果该`go.sum`文件不存在，或者该文件不包含所下载文件的哈希，则该`go`命令可以使用[校验和数据库](https://golang.org/ref/mod#checksum-database)来验证该哈希，[校验和数据库](https://golang.org/ref/mod#checksum-database)是公共可用模块的哈希的全局源。验证散列后，该`go`命令会将其`go.sum`添加到模块缓存中，并将下载的文件添加到模块缓存中。如果模块是私有的（通过`GOPRIVATE`或`GONOSUMDB`环境变量匹配），或者如果禁用了校验和数据库（通过设置`GOSUMDB=off`），则该`go`命令将接受哈希并将该文件添加到模块高速缓存中，而不进行验证。

模块缓存通常由系统上的所有Go项目共享，并且每个模块可能具有其自己的`go.sum`文件，这些文件可能具有不同的哈希值。为避免需要信任其他模块，该`go`命令`go.sum`在访问模块缓存中的文件时会使用主模块的哈希值进行验证。压缩文件哈希的计算成本很高，因此该`go`命令检查与压缩文件一起存储的预先计算的哈希，而不是重新哈希文件。该[`go mod verify`](https://golang.org/ref/mod#go-mod-verify)命令可用于检查zip文件和提取的目录，因为它们已被添加到模块高速缓存中，所以没有被修改。

### go.sum文件

模块`go.sum`的根目录中可能会在文件旁边有一个名为的文本`go.mod`文件。该`go.sum`文件包含模块直接和间接依赖关系的加密哈希。当`go`命令将模块`.mod`或`.zip`文件下载到[模块高速缓存中时](https://golang.org/ref/mod#module-cache)，它将计算哈希值并检查该哈希值是否与主模块`go.sum`文件中的相应哈希值匹配。`go.sum`如果模块没有依赖项，或者使用[`replace`指令将](https://golang.org/ref/mod#go-mod-file-replace)所有依赖项替换为本地目录，则可能为空或不存在 。

每行中的`go.sum`三个字段均由空格分隔：模块路径，版本（可能以结尾`/go.mod`）和哈希。

- 模块路径是哈希所属的模块的名称。
- 版本是哈希所属的模块的版本。如果版本以结尾`/go.mod`，则哈希仅用于模块`go.mod`文件；否则，仅用于模块文件。否则，哈希用于模块`.zip`文件中的文件。
- 哈希列由算法名称（如`h1`）和以冒号（`:`）分隔的base64编码的密码哈希组成。当前，SHA-256（`h1`）是唯一受支持的哈希算法。如果将来会发现SHA-256中的漏洞，则会添加对另一种算法（命名`h2`，依此类推）的支持。

该`go.sum`文件可能包含模块多个版本的哈希。该`go` 命令可能需要`go.mod`从一个依赖项的多个版本中加载文件，以便执行[最少的版本选择](https://golang.org/ref/mod#minimal-version-selection)。 `go.sum`可能还包含不再需要的模块版本的哈希值（例如，升级后）。[`go mod tidy`](https://golang.org/ref/mod#go-mod-tidy)会添加缺少的哈希，并会从中删除不必要的哈希`go.sum`。

### 校验和数据库

校验和数据库是行的全局源`go.sum`。该`go`命令在许多情况下都可以使用它来检测代理服务器或原始服务器的不当行为。

校验和数据库允许所有公开可用的模块版本具有全局一致性和可靠性。这使得不受信任的代理成为可能，因为如果不注意它就无法提供错误的代码。它还可以确保与特定版本关联的位不会在一天之间变化，即使该模块的作者随后更改了其存储库中的标签也是如此。

校验和数据库由[sum.golang.org提供服务](https://sum.golang.org/)，该文件由Google运行。它是线哈希的[透明日志](https://research.swtch.com/tlog)（或“ Merkle树”），`go.sum`由[Trillian](https://github.com/google/trillian)支持。Merkle树的主要优点是独立的审核员可以验证它是否未被篡改，因此它比简单的数据库更值得信赖。

该`go`命令使用最初在[提案：保护Public Go Module生态系统中](https://go.googlesource.com/proposal/+/master/design/25530-sumdb.md#checksum-database)概述的协议与校验和数据库进行交互。

下表指定了校验和数据库必须响应的查询。对于每个路径，`$base`是校验和数据库URL的路径部分， `$module`是模块路径，并且`$version`是版本。例如，如果校验和数据库URL为`https://sum.golang.org`，并且客户端正在请求该模块`golang.org/x/text`的版本记录`v0.3.2`，则客户端将发送的`GET`请求 `https://sum.golang.org/lookup/golang.org/x/text@v0.3.2`。

为了避免在不区分大小写的文件系统中提供服务时产生歧义，将元素`$module`和`$version`元素进行 [大小写编码，方法](https://pkg.go.dev/golang.org/x/mod/module#EscapePath) 是将每个大写字母替换为感叹号，后跟相应的小写字母。这允许模块`example.com/M`与 `example.com/m`既存储在磁盘上，因为前者被编码为 `example.com/!m`。

路径中被方形制动器包围的部分，例如`[.p/$W]`表示可选值。

| 路径                            | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| `$base/latest`                  | 返回最新日志的签名的编码树描述。此签名的描述采用[便笺](https://pkg.go.dev/golang.org/x/mod/sumdb/note)的形式， 即已由一个或多个服务器密钥签名的文本，可以使用服务器的公共密钥进行验证。树描述提供了树的大小以及树头的哈希值。在中描述了这种编码 `golang.org/x/mod/sumdb/tlog#FormatTree`。 |
|                                 |                                                              |
| `$base/lookup/$module@$version` | 返回关于`$module` at的条目的日志记录编号`$version`，然后返回该记录的数据（即at 的`go.sum`行）以及包含该记录的带符号的已编码树描述。 `$module``$version` |
|                                 |                                                              |
| `$base/tile/$H/$L/$K[.p/$W]`    | 返回一个[log tile]（https://research.swtch.com/tlog#serving_tiles），它是组成日志一部分的一组哈希。每个图块在图块级别`$L`（`$K`从左数th处）的二维坐标中定义， 图块高度为 `$H`。可选的`.p/$W`后缀表示仅带有`$W`哈希值的部分日志图块。如果找不到局部图块，则客户端必须回退到获取完整图块。 |
|                                 |                                                              |
| `$base/tile/$H/data/$K[.p/$W]`  | 返回叶哈希值的记录数据 `/tile/$H/0/$K[.p/$W]`（带有文字`data`路径元素）。 |
|                                 |                                                              |

如果`go`命令查询校验和数据库，则第一步是通过`/lookup`端点检索记录数据。如果模块版本尚未记录在日志中，则校验和数据库将在回复之前尝试从原始服务器获取它。该`/lookup`数据提供了此模块版本的总和以及它在日志中的位置，该信息通知客户端应获取哪些图块以进行打样。`go`在向`go.sum`主模块的`go.sum`文件中添加新行之前，该命令执行“包含”证明（日志中存在特定记录）和“一致性”证明（树未被篡改）。重要的是来自`/lookup` 除非首先针对签名树哈希对它进行身份验证，然后根据客户端的签名树哈希的时间轴对签名树哈希进行身份验证，否则切勿使用。

校验和数据库提供的签名树哈希和新的图块存储在模块高速缓存中，因此该`go`命令仅需要获取丢失的图块。

该`go`命令不需要直接连接到校验和数据库。它可以通过[镜像校验和数据库](https://go.googlesource.com/proposal/+/master/design/25530-sumdb.md#proxying-a-checksum-database) 并支持上述协议的模块代理来请求模块总和。这对于阻止组织外部的请求的私人公司代理特别有用。

在`GOSUMDB`环境变量识别校验数据库的名称使用和可选的公钥和URL，如：

```
GOSUMDB="sum.golang.org"
GOSUMDB="sum.golang.org+<publickey>"
GOSUMDB="sum.golang.org+<publickey> https://sum.golang.org"
```

该`go`命令知道的公钥`sum.golang.org`，并且该名称`sum.golang.google.cn`（在中国大陆内部可用）连接到 `sum.golang.org`校验和数据库。使用任何其他数据库都需要显式提供公钥。URL默认为`https://`后跟数据库名称。

`GOSUMDB`默认为`sum.golang.org`，由Google运行Go校验和数据库。有关服务的隐私权政策，请参阅https://sum.golang.org/privacy。

如果`GOSUMDB`将`off`if设置为，或者如果`go get`使用该`-insecure` 标志调用，则将不查询校验和数据库，并且接受所有无法识别的模块，但其代价是放弃了对所有模块的已验证可重复下载的安全保证。绕过特定模块的校验和数据库的更好方法是使用`GOPRIVATE`或`GONOSUMDB` 环境变量。有关详细信息，请参见[专用模块](https://golang.org/ref/mod#private-modules)。

该`go env -w`命令可用于 为将来的命令调用[设置这些变量](https://golang.org/pkg/cmd/go/#hdr-Print_Go_environment_information)`go`。

## 环境变量

`go`可以使用下面列出的环境变量来配置命令中的模块行为。该列表仅包括与模块相关的环境变量。请参阅[`go help environment`](https://golang.org/cmd/go/#hdr-Environment_variables)以获取`go`命令可识别的所有环境变量的列表。

| 变量          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `GO111MODULE` | 控制`go`命令以模块感知模式还是`GOPATH`模式运行。可以识别三个值：`off`：该`go`命令将忽略 `go.mod`文件并以`GOPATH`模式运行。`on`：`go`即使没有`go.mod`文件，该命令也以模块感知模式运行。`auto`（或未设置）：`go`如果`go.mod`当前目录或任何父目录中存在文件，则该命令以模块感知模式运行（默认行为）。有关更多信息，请参见支持[模块的命令](https://golang.org/ref/mod#mod-commands)。 |
| `GOMODCACHE`  | 该`go`命令将存储下载的模块和相关文件的目录。有关此目录的结构的详细信息，请参见[模块缓存](https://golang.org/ref/mod#module-cache)。如果`GOMODCACHE`未设置，则默认为 `$GOPATH/pkg/mod`。 |
| `GOINSECURE`  | 逗号分隔[`path.Match`](https://golang.org/pkg/path/#Match)的模块路径前缀（以Go的语法）的列表 ，可能总是以不安全的方式获取。仅适用于直接获取的依赖项。与上的`-insecure`标志不同`go get`， `GOINSECURE`它不会禁用模块校验和数据库验证。`GOPRIVATE`或`GONOSUMDB`可以用来实现这一目标。 |
| `GONOPROXY`   | 逗号分隔[`path.Match`](https://golang.org/pkg/path/#Match)的模块路径前缀（以Go的语法）列表 ，应始终从版本控制存储库而不是从模块代理直接获取。如果`GONOPROXY`未设置，则默认为 `GOPRIVATE`。请参阅 [隐私](https://golang.org/ref/mod#private-module-privacy)。 |
| `GONOSUMDB`   | 逗号分隔[`path.Match`](https://golang.org/pkg/path/#Match)的模块路径前缀的全局模式列表（按Go的语法 ），`go`不应使用校验和数据库验证其校验和。如果`GONOSUMDB`未设置，则默认为 `GOPRIVATE`。请参阅 [隐私](https://golang.org/ref/mod#private-module-privacy)。 |
| `GOPATH`      | 在`GOPATH`模式下，该`GOPATH`变量是可能包含Go代码的目录的列表。在模块感知模式下，[模块缓存](https://golang.org/ref/mod#glos-module-cache)存储在`pkg/mod`第一个`GOPATH`目录的子目录中。缓存外部的模块源代码可以存储在任何目录中。如果`GOPATH`未设置，则默认为`go` 用户主目录的子目录。 |
| `GOPRIVATE`   | 以逗号分隔[`path.Match`](https://golang.org/pkg/path/#Match)的模块路径前缀的全局模式列表（按Go的语法 ），应视为私有。`GOPRIVATE` 是默认值`GONOPROXY`和 `GONOSUMDB`。`GOPRIVATE`本身没有其他含义。请参阅[隐私](https://golang.org/ref/mod#private-module-privacy)。 |
| `GOPROXY`     | 模块代理URL的列表，以逗号（`,`）或管道（`|`）分隔。当`go`命令查找有关模块的信息时，它将顺序联系列表中的每个代理，直到收到成功的响应或终端错误。代理可能会以404（未找到）或410（已消失）状态进行响应，以指示该模块在该服务器上不可用。该`go`命令的错误回退行为由URL之间的分隔符决定。如果代理URL后跟逗号，则该`go`命令会在404或410错误后退回到下一个URL；否则，该命令将返回到下一个URL。所有其他错误均视为终端错误。如果代理URL后跟管道，则`go`在发生任何错误（包括非HTTP错误，例如超时）后，该命令将退回到下一个来源。`GOPROXY`网址可能具有的方案`https`， `http`或`file`。如果URL没有方案， `https`则假定为。可以将模块缓存直接用作文件代理：`GOPROXY =文件：// $（转到环境GOMODCACHE）/缓存/下载`可以使用两个关键字代替代理URL：`off`：禁止从任何来源下载模块。`direct`：直接从版本控制存储库下载，而不使用模块代理。`GOPROXY`默认为 `https://proxy.golang.org,direct`。在该配置下，该`go`命令首先与Google运行的Go模块镜像联系，然后在镜像没有模块的情况下退回到直接连接。有关 镜像的隐私政策，请参阅 https://proxy.golang.org/privacy。该`GOPRIVATE`和 `GONOPROXY`环境变量可以设置为防止使用代理服务器下载特定的模块。有关私有代理配置的信息，请参阅 [隐私](https://golang.org/ref/mod#private-module-privacy)。有关如何使用代理的更多信息，请参见[模块代理](https://golang.org/ref/mod#module-proxy)和 [将程序包解析为模块](https://golang.org/ref/mod#resolve-pkg-mod)。 |
| `GOSUMDB`     | 标识要使用的校验和数据库的名称，并可选地标识其公共密钥和URL。例如：`GOSUMDB =“ sum.golang.org” GOSUMDB =“ sum.golang.org + <publickey>” GOSUMDB =“ sum.golang.org + <publickey> https://sum.golang.org `该`go`命令知道的公钥， `sum.golang.org`并且名称 `sum.golang.google.cn`（在中国大陆内部可用）连接到`sum.golang.org`数据库。使用任何其他数据库都需要显式提供公钥。URL默认为`https://`后跟数据库名称。`GOSUMDB`默认为`sum.golang.org`，由Google运行Go校验和数据库。有关 服务的隐私权政策，请参阅 https://sum.golang.org/privacy。如果`GOSUMDB`将`off`if设置为或 `go get`用`-insecure`标志调用，则不查询校验和数据库，并且接受所有无法识别的模块，但其代价是放弃了对所有模块的已验证可重复下载的安全保证。绕过特定模块的校验和数据库的更好方法是使用 `GOPRIVATE`或`GONOSUMDB`环境变量。有关更多信息，请参见[身份验证模块](https://golang.org/ref/mod#authenticating)和 [隐私](https://golang.org/ref/mod#private-module-privacy)。 |

## 词汇表

**构建约束：**确定编译软件包时是否使用Go源文件的条件。构建约束可以用文件名后缀（例如，`foo_linux_amd64.go`）或构建约束注释（例如，`// +build linux,amd64`）来表示。请参阅[构建约束](https://golang.org/pkg/go/build/#hdr-Build_Constraints)。

**构建列表：**将被用于生成命令如模块版本列表`go build`，`go list`或`go test`。使用[最小的版本选择，](https://golang.org/ref/mod#glos-minimal-version-selection)从[主模块的](https://golang.org/ref/mod#glos-main-module) [`go.mod` 文件](https://golang.org/ref/mod#glos-go-mod-file)和可`go.mod`传递需要的模块中[的](https://golang.org/ref/mod#glos-main-module)[文件](https://golang.org/ref/mod#glos-go-mod-file)确定构建列表。构建列表包含[模块图中](https://golang.org/ref/mod#glos-module-graph)所有模块的版本，而不仅仅是与特定命令相关的版本。

**规范版本：**格式正确的[版本，](https://golang.org/ref/mod#glos-version)没有`+incompatible`。例如，`v1.2.3` 是规范版本，但`v1.2.3+meta`不是。

**`go.mod`文件：**定义模块的路径，要求和其他元数据的文件。出现在[模块的根目录中](https://golang.org/ref/mod#glos-module-root-directory)。请参阅[`go.mod` 文件](https://golang.org/ref/mod#go-mod-file)部分。

**导入路径：**用于在Go源文件中导入包的字符串。[包路径的](https://golang.org/ref/mod#glos-package-path)同义词。

**主模块：**在其中`go`调用命令的模块。

**主版本：**语义版本（`1`中的第一个数字`v1.2.3`）。在具有不兼容更改的发行版中，必须增加主版本，并且次要版本和修补程序版本必须设置为0。主版本0的语义版本被认为是不稳定的。

**主版本子目录：**版本控制存储库中与模块的[主版本后缀](https://golang.org/ref/mod#glos-major-version-suffix)匹配的子目录，可以在其中定义模块。例如，可以在存储库根目录或主版本子目录中定义`example.com/mod/v2`具有[根路径](https://golang.org/ref/mod#glos-repository-root-path) 的存储库中的模块。请参见[存储库中的模块目录](https://golang.org/ref/mod#vcs-dir)。`example.com/mod``v2`

**主版本后缀：**与主版本号匹配的模块路径后缀。例如，`/v2`在中`example.com/mod/v2`。在`v2.0.0`或更高版本中需要主版本后缀，而在较早版本中则不允许。请参阅[主要版本后缀部分](https://golang.org/ref/mod#major-version-suffixes)。

**最小版本选择（MVS）：**用于确定将在构建中使用的所有模块的版本的算法。有关详细信息，请参见“[最小版本选择](https://golang.org/ref/mod#minimal-version-selection)”部分 。

**次要版本：**语义版本中的第二个数字（`2`in `v1.2.3`）。在具有新的，向后兼容功能的发行版中，必须增加次要版本，并且修补程序版本必须设置为0。

**模块：**一起发布，版本控制和分发的软件包的集合。

**模块缓存：**位于中的本地目录，用于存储下载的模块 `GOPATH/pkg/mod`。请参阅[模块缓存](https://golang.org/ref/mod#module-cache)。

**模块图：**模块需求的有向图，植根于[主模块](https://golang.org/ref/mod#glos-main-module)。图中的每个顶点都是一个模块。每个边都是文件中的`require`语句的版本`go.mod`（主模块文件中的`replace`和 `exclude`语句均受此约束）`go.mod`。

**模块路径：**标识模块并充当模块内包导入路径的前缀的路径。例如，`"golang.org/x/net"`。

**模块代理：**实现[`GOPROXY` 协议的](https://golang.org/ref/mod#goproxy-protocol)Web服务器。该`go`命令`go.mod`从模块代理下载版本信息， 文件和模块zip文件。

**模块根目录：**包含`go.mod`定义模块的文件的目录。

**模块子目录：**[模块路径](https://golang.org/ref/mod#glos-module-path)在[存储库根路径](https://golang.org/ref/mod#glos-repository-root-path)之后的部分，指示在其中定义模块的子目录。如果为非空，则模块子目录也是[语义版本标记](https://golang.org/ref/mod#glos-semantic-version-tag)的前缀。即使模块位于[主版本子目录中](https://golang.org/ref/mod#glos-major-version-subdirectory)，模块子目录也不会包含 [主版本后缀](https://golang.org/ref/mod#glos-major-version-suffix)（即使有）。请参阅[模块路径](https://golang.org/ref/mod#module-path)。

**软件包：**同一目录中一起编译的源文件的集合。请参阅Go语言规范中的“[包”部分](https://golang.org/ref/spec#Packages)。

**包路径：**唯一标识包的路径。包路径是与[模块](https://golang.org/ref/mod#glos-module-path)内的子目录连接的[模块路径](https://golang.org/ref/mod#glos-module-path)。例如，子目录中`"golang.org/x/net/html"`模块`"golang.org/x/net"`中软件包的软件包路径`"html"`。[导入路径的](https://golang.org/ref/mod#glos-import-path)同义词 。

**补丁版本：**在语义的版本（第三数`3`中`v1.2.3`）。在不更改模块公共接口的发行版中，必须增加补丁程序版本。

**预发行版本：**补丁版本之后紧跟短划线的版本，其后是一系列以点分隔的标识符 `v1.2.3-beta4`。预发行版本被认为是不稳定的，并且不假定与其他版本兼容。预发行版本在相应发行版本之前排序：`v1.2.3-pre`在之前`v1.2.3`。另请参见 [发行版本](https://golang.org/ref/mod#glos-release-version)。

**伪版本：**对版本标识符（例如Git提交哈希）和来自版本控制系统的时间戳进行编码的版本。例如， `v0.0.0-20191109021931-daa7c04131f5`。用于[与非模块存储库兼容，](https://golang.org/ref/mod#non-module-compat)以及在其他情况下（如果没有标记版本）。

**发布版本：**不包含预发布后缀的版本。例如 `v1.2.3`，不是`v1.2.3-pre`。另请参见[预发行版本](https://golang.org/ref/mod#glos-pre-release-version)。

**信息库根路径：**[模块路径](https://golang.org/ref/mod#glos-module-path)中与版本控制信息库根目录相对应的部分。请参阅[模块路径](https://golang.org/ref/mod#module-path)。

**语义版本标记：**在一个映射一个版本控制库的标签 [版本](https://golang.org/ref/mod#glos-version)到一个特定的版本。请参见将[版本映射到提交](https://golang.org/ref/mod#vcs-version)。

**供应商目录：**一个名为目录`vendor`，其中包含在主模块中构建软件包所需的其他模块的软件包。保持与 [`go mod vendor`](https://golang.org/ref/mod#go-mod-vendor)。请参阅[供应商](https://golang.org/ref/mod#vendoring)。

**版本：**模块的不变快照的标识符，写为字母，`v`后跟语义版本。请参阅“[版本](https://golang.org/ref/mod#versions)”部分 。

![地鼠](https://golang.org/lib/godoc/images/footer-gopher.jpg)

- [版权](https://golang.org/doc/copyright.html)
-  

- [服务条款](https://golang.org/doc/tos.html)
-  

- [隐私政策](http://www.google.com/intl/en/policies/privacy/)
-  

- [报告网站问题](http://golang.org/issues/new?title=x/website:)

[受Google支持](https://google.com/)


