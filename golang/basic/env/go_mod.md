# go mod

## 介绍

**使用 GOPATH 问题**

- 代码开发必须在 GOPATH 的 src 目录下，不然，就有问题；
- 依赖手动管理；
- 依赖包没有版本可言。

**使用 vendor 问题**

- 解决了包依赖，一个配置文件就管理；

- 依赖包全都下载到项目 vendor 下，每个项目都把有一份。

因此，golang 1.11 后新加特性 go modules。

Modules官方定义为：

> 模块是相关 Go 包的集合。modules 是源代码交换和版本控制的单元。go 命令直接支持使用 modules，包括记录和解析对其他模块的依赖性。modules 替换旧的基于 GOPATH 的方法来指定在给定构建中使用哪些源文件。

## 安装

首先， 安装 go 版本必须大于 1.11，安装过程可参照官网，此处不再赘述。

**查看 go 版本**

```shell
go version
```

**设置 go mod 和 go proxy**

```shell
go env -w GOBIN=/usr/local/go/bin
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

> GO111MODULE 有三个值：off, on 和 auto。
>
> GO111MODULE=off，go 命令行将不会支持 module 功能，寻找依赖包的方式将会沿用旧版本那种通过 vendor 目录或者 GOPATH模式来查找。
>
> GO111MODULE=on，go 命令行会使用 modules，而一点也不会去 GOPATH 目录下查找。
>
> GO111MODULE=auto，默认值，go 命令行将会根据当前目录来决定是否启用 module 功能。这种情况下可以分为两种情形：
>
> - 当前目录在 $GOPATH/src 之外且该目录包含 go.mod 文件；
> - 当前文件在包含 go.mod 文件的目录下面。

当 modules 功能启用时，依赖包的存放位置变更为 $GOPATH/pkg，允许同一个 package 多个版本并存，且多个项目可以共享缓存的 module。

## 使用

```shell
Usage:

    go mod <command> [arguments]

The commands are:

    download    download modules to local cache（下载依赖包）
    edit        edit go.mod from tools or scripts（编辑 go.mod）
    graph       print module requirement graph（打印模块依赖图）
    init        initialize new module in current directory（再当前目录初始化 mod）
    tidy        add missing and remove unused modules（拉取缺少模块，移除不用的模块）
    vendor      make vendored copy of dependencies（将依赖复制 vender 下）
    verify      verify dependencies have expected content（验证依赖正确性）
    why         explain why packages or modules are needed（解释依赖）
```

### 初始化新项目

**初始化项目**

```shell
mkdir ${PROJECT_NAME}
cd ${PROJECT_NAME}
go mod init ${PROJECT_NAME}
```

**查看 go.mod 文件**

```text
module ${PROJECT_NAME}

go 1.17
```

go.mod 文件一旦创建后，它的内容将会被 go toolchain 全面掌控。go toolchain 会在各类命令执行时，比如 go get、go build、go mod 等修改和维护 go.mod 文件。

> go.mod 提供了四个命令：
>
> - module 语句指定包的名字（路径）；
> - require 语句指定的依赖项模块；
> - replace 语句可以替换依赖项模块；
> - exclude 语句可以忽略依赖项模块。

**添加依赖**

```go
package main

import (
    "github.com/go-basic/uuid"
)
```

**执行 go run 命令，再查看 go.mod 文件**

```text
module Demo

go 1.17

require github.com/go-basic/uuid v1.0.0
```

go mod 安装 package 的原则是先拉最新的 release tag，若无 tag 则拉最新的 commit。go 会自动生成一个 go.sum 文件来记录 dependency tree。

可以使用命令 go list -m -u all 来检查可以升级的 package，使用 go get -u need-upgrade-package 升级后会将新的依赖版本更新到 go.mod（也可以使用 go get -u 升级所有依赖）。

### 管理项目中包

**go get 升级包**

- 运行 go get -u 将会升级到最新的次要版本或者修订版本（x.y.z, z是修订版本号， y是次要版本号）；
- 运行 go get -u=patch 将会升级到最新的修订版本；
- 运行 go get package@version 将会升级到指定的版本号 version；
- 运行 go get 如果有版本的更改，那么 go.mod 文件也会更改；

**使用 replace 替换无法直接获取的 package**

由于某些已知的原因，并不是所有的 package 都能成功下载。modules 可以通过在 go.mod 文件中使用 replace 指令替换成 github 上对应的库。

```text
replace (
    golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
)
```

### 发布与使用

**创建模块**

首先，我们按照上面的步骤随意创建一个模块 demo1，并将其上传到 github。

```shell
# 创建模块步骤省略,以下为上传github步骤
git init
vim .gitiiignore
git commit -am "init"
git remote add origin git@github.com:xxx/demo1.git
git push -u origin master
```

可以通过如下命令使用：

```text
go get github.com/xxx/demo1
```

这个时候没有加 tag，所以，没有版本的控制。默认是 v0.0.0 后面接上时间和 commitid。如：

```text
demo1@v0.0.0-20200517004046-ee882713fd1e
```

不过不建议这样做，因为没有进行版本控制管理。

**添加模块版本号**

```shell
git tag v1.0.0
git push --tags
```

操作完，我们的 module 就发布了一个 v1.0.0 的版本。

推荐在这个状态下，再切出一个分支，用于后续 v1.0.0 的修复推送,不要直接在 master 分支修复。

```shell
git checkout -b v1
git push -u origin v1
```

**在项目中使用模块**

main.go：

```go
package main

import (
    "fmt"
    "github.com/xxx/demo1"
)
```

生成 go.mod，并查看：

```text
module Gone

go 1.14

require (
    github.com/xxx/demo1 v1.0.0
)
```

**修复模块 bug 并提交**

假如我们在 v1 版本上修复模块的一个 bug，然后 push 到 github：

```shell
git commit -m "xxx" xxx.go
git tag v1.0.1
git push --tags origin v1
```

**更新项目中模块**

```shell
go get -u
go get -u=patch
go get github.com/xxx/demo1@v1.0.1
```

go.mod 文件更新为：

```
module Gone

go 1.14

require (
    github.com/xxx/demo1 v1.0.1
)
```

**更新模块主版本**

```shell
git commit xxx.go -m "xxx"
git checkout -b v2 # 用于 v2 版本，后续修复 v2
git commit go.mod -m "Bump version to v2"
git tag v2.0.0
git push --tags origin v2 
```

**项目中同时使用 v2 版本模块**

即使发布了库的新不兼容版本，现有软件也不会中断，因为它将继续使用现有版本1.0.1。go get -u 将不会获得版本2.0.0。如果想使用v2.0.0,代码改成如下：

```go
package main

import (
    "fmt"
    "github.com/xxx/demo1"
    demov2 "github.com/xxx/demo1/v2"
)
```

### vendor

默认是忽略 vendor 的，如果想在项目目录下有 vendor 可以执行下面命令：

```shell
go vendor
```

当然，如果构建程序的时候，希望使用vendor中的依赖:

```shell
go build -mod vendor
```

