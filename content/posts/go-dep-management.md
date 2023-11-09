+++
title = "Go Dep Management"
date = "2023-11-09T18:07:39+08:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

`Go Modules`是Go 语言从 1.11 版本之后官方推出的**版本管理**工具。1.11+以后默认`GO111MODULE=auto`，只要当前目录在`$GOPATH/src`之外且该目录包含go.mod文件就会使用`Go Modules`。

使用modules依赖管理后：

-   不需要将代码再放入$GOPATH/src
-   对于已经转移的包，可以用replace在go.mod文件中替换，不需要修改代码

当modules 功能启用时，依赖包的存放位置为`$GOPATH/pkg`，允许同一个package多个版本并存，且多个项目可以共享缓存的 module。

## go.mod

### `go mod`命令

| 命令 | 说明 |
| --- | --- |
| download | download modules to local cache(下载依赖包到本地缓存) |
| edit | edit go.mod from tools or scripts (编辑go.mod) |
| graph | print module requirement graph (打印模块依赖图) |
| init | initialize new module in current directory (在当前目录初始化go.mod) |
| tidy | add missing and remove unused modules(拉取缺少的模块，移除不用的模块) |
| vendor | make vendored copy of dependencies(将依赖复制到vendor下) |
| verify | verify dependencies have expected content (验证依赖是否正确) |
| why | explain why packages or modules are needed(解释为什么需要依赖) |

执行 `go get`命令，在下载依赖包的同时还可以指定依赖包的版本

-   运行`go get -u`命令会将项目中的包升级到最新的次要版本或者修订版本
-   运行`go get -u=patch`命令会将项目中的包升级到最新的修订版本
-   运行`go get [包名]@[版本号]`命令会下载对应包的指定版本或者将对应包升级到指定的版本

当然我们也可以先不`go get`，在最后`go run main.go`时，`go.mod`同样会自动查找依赖自动下载

### go.mod关键字

| 命令字 | 作用 |
| --- | --- |
| module | 指定包的名字（路径） |
| require | 指定依赖项模块 |
| replace | 替换依赖项模块 |
| exclude | 忽略依赖项模块 |

## go.sum

go.sum就是依据hash来检测下载下来的依赖，是不是和该版本依赖复合

## `version`的确定规则

一、项目是否打tag？

如果项目没有打 tag，会生成一个版本号，格式如下：  
v0.0.0-commit日期-commitID

比如 `v0.0.0-20180321164747-3a771d992973`。

引用一个项目的特定分支，比如 develop branch，也会生成类似的版本号：  
v当前版本+1-commit日期-commitID

比如 `v1.3.4-0.20191205000432-012d92843b00`。

二、项目有没有用 go module？

如果项目有用到 go module，那么就是正常地用 tag 来作为版本号。

比如 `github.com/DATA-DOG/go-sqlmock v1.3.3 h1:CWUqKXe0s8A2z6qCgkP4Kru7wC11YoAnoupUKFDnH08=`。

如果项目打了 tag，但是没有用到 go module，为了跟用了 go module 的项目相区别，需要加个 `+incompatible` 的标志。

比如 `github.com/google/martian v2.1.0+incompatible/go.mod h1:9I4somxYTbIHy5NJKHRl3wXiIaQGbYVAs8BPL6v8lEs=`

三、项目用的 go module 版本是不是 v2+？

关于 go module v2+ 的特性，可以参考 Go 的官方文档：[blog.golang.org/v2-go...](https://link.juejin.cn/?target=https%3A%2F%2Fblog.golang.org%2Fv2-go-modules "https://link.juejin.cn?target=https%3A%2F%2Fblog.golang.org%2Fv2-go-modules")。简单而言，就是通过让依赖路径带版本号后缀来区分同一个项目里不同版本的依赖，类似于 `gopkg.in/xxx.v2` 的效果。

对于使用了 v2+ go module 的项目，项目路径会有个版本号的后缀。

比如 `github.com/googleapis/gax-go/v2 v2.0.5 h1:sjZBwGj9Jlw33ImPtvFviGYvseOtDM7hkSKB7+Tv3SM=`

## go.work

随着go 1.18版本的正式发布，多模块工作区也被引入（WorkSpaces），多模块工作区能够使开发者能够更容易地同时处理多个模块的工作， 如：方便进行依赖的代码调试(打断点、修改代码)、排查依赖代码 bug 。方便同时进行多个仓库/模块并行开发调试

#### `go work`支持命令

-   `go work init` 初始化工作区文件，用于生成go.work工作区文件
-   `go work use`添加新的模块到工作区

> go work use ./example 添加一个模块到工作区
> 
> go work use ./example ./example1 添加多个模块到工作区
> 
> go work use -r ./example 递归./example目录到当前工作区
> 
> 删除命令: go work edit -dropuse=./example功能

-   `go work sync`将工作区的构建列表同步到工作区模块
-   `go env GOWORK`查看环境变量，查看当前工作区文件路径，可以排查工作区文件是否设置正确，go.work路径找不到可以使用GOWORK指定

#### go.work文件结构

go.work文件结构和go.mod文件结构类似，支持Go版本号、指定工作区和需要替代的仓库

`use`指定使用的模块目录，可以使用`go work use`添加模块，也可以手动修改go.work工作区添加新的模块，在工作区中添加了模块路径，编译时会自动使用`use`中的本地代码进行编译

`replaces`替换依赖仓库地址，`replaces`命令与go.mod指令相同，用于替换项目中依赖的仓库地址，需要注意的是，`replaces`和`use`不能同时指定相同的本地代码路径

#### go.work文件优先级高于go.mod

在同时使用go.work和go.mod的`replaces`功能时，分别指定不同的代码仓库路径，go.work优先级高于go.mod中定义

```
//go.mod 中定义替换为本地仓库 example
replace ( 
    github.com/link1st/example => ./example 
)

//go.work 中定义替换为本地仓库 example1
replace ( 
    github.com/link1st/example => ./example1 
)
```

在代码构建时，使用的是go.work指定的example1仓库代码，go.work优先级别更高

使用go.work时，不同的项目文件必须在有同一个根目录， 推荐在`$GOPATH`路径下执行，生成go.work文件

## 参考文档

[掘金文章](https://juejin.cn/post/7182091980099289147)

[Go Modules官方文档](https://learnku.com/docs/go-mod/1.17)