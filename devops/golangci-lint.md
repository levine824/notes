# Golang 有哪些 lint 工具

作为 Gopher，我们有官方提供的 go fmt 工具能够完成基础的格式调整（当然，这远远不够），开源社区里提供的各种 linter 还是非常多，而且有效的。

这里也推荐一个 linter 列表 (具体参考[1]) ：

- gofmt：Go SDK 自带的代码格式检查工具，用于检查缩进、文件尾是否有空行等。go fmt 命令（即：gofmt -l -w）可使用该工具格式化代码；
- go vet：Go SDK 自带的可疑代码检查工具，检查 unsafe.Pointer 的错误使用、unreacheable code 等可能有问题的代码；
- golangci-lint：社区提供的 linters 聚合运行工具，内置了多个 linter（代码检查工具），支持配置化 & 并行运行。

具体的 lint 规则非常多，而且是一个动态增加的过程，作为业务研发，我们希望能有一个统一的载体，让我们可以直接用一个 linter 就能运行各种规则，并且要可配置。这方面业界公认最出色的还是 golangci-lint，下面我们就来了解一下。

# golangci-lint

golangci-lint 是一个 Go linters aggregator，并不是一个 linter，其作用在于将每个独立的 linter 聚合起来得到更好的体验和满足不同的需求。

特色功能：

- 并行跑各个 linter，重复利用 Golang 的构建缓存，并缓存分析结果；
- 提供 yaml 配置，自定义你需要的 linter 以及规则；
- 跟 VSCode，Sublimt Text, Golang, Emacs, Vim, GitHub Actions 进行集成，你可以开箱即用；
- 自带常用 linter，无需开发者再次安装；
- 输出的问题有颜色区分，标识源码所在行以及出问题的标识符。

感兴趣的同学也可以参考官方文档 (具体参考[2])。

# 安装

```shell
# binary will be $(go env GOPATH)/bin/golangci-lint
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin 
```

# 实战

```shell
golangci-lint run // 等价于 golangci-lint run ./...
golangci-lint run dir1 dir2/... dir3/file1.go // 选择部分目录或者文件执行
golangci-lint linters // 查看开启的 linter
```

如果我们希望自己来指定规则，而不是走默认规则，需要怎么配置呢？

首先，在你的项目根目录添加配置文件，用下面哪一个后缀都ok：

- .golangci.yml
- .golangci.yaml

配置文件的格式如下：

```yaml
# Options for analysis running.
run:
  # See the dedicated "run" documentation section.
  option: value
# output configuration options
output:
  # See the dedicated "output" documentation section.
  option: value
# All available settings of specific linters.
linters-settings:
  # See the dedicated "linters-settings" documentation section.
  option: value
linters:
  # See the dedicated "linters" documentation section.
  option: value
issues:
  # See the dedicated "issues" documentation section.
  option: value
severity:
  # See the dedicated "severity" documentation section.
  option: value
```

这里我们主要关注三个：

- run：控制的是lint运行的选项，如并发数，超时时间，是否包含test文件，需要忽略的路径，文件，Golang 版本；
- linters：选择你希望启用的linter，示例：

```yaml
linters:
  # Disable all linters.
  # Default: false
  disable-all: true
  # Enable specific linter
  # https://golangci-lint.run/usage/linters/#enabled-by-default-linters
  enable:
    - gocognit
    - goconst
    - gocritic
    - gocyclo
    - godot
    - godox
    - goerr113
    - gofmt
    - gofumpt
    - tenv
    - whitespace
    - wrapcheck
    - wsl
  # Enable all available linters.
  # Default: false
  enable-all: true
  # Disable specific linter
  # https://golangci-lint.run/usage/linters/#disabled-by-default-linters--e--enable
  disable:
    - asasalint
    - asciicheck
    - bidichk
    - bodyclose
    - containedctx
    - contextcheck
    - cyclop
    - deadcode
    - decorder
    - wsl
  # Enable presets.
  # https://golangci-lint.run/usage/linters
  presets:
    - bugs
    - comment
    - complexity
    - test
    - unused
  # Run only fast linters from enabled linters set (first run won't be fast)
  # Default: false
  fast: true
```

- linters-settings：具体每个 linter 的配置，类似我们上面对 gocyclo 配置最低圈复杂度。

我们在工作路径增加 .golangci.yml ，配置自定义的规则。

```yaml
linters:
  # Disable all linters.
 # Default: false
disable-all: true
  # Enable specific linter
 # https://golangci-lint.run/usage/linters/#enabled-by-default-linters
enable:
    # default linter
- deadcode
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - typecheck
    - unused
    - varcheck
    # 新增 linter
- gocyclo
    - gofmt
    - goimports
  # Run only fast linters from enabled linters set (first run won't be fast)
 # Default: false
fast: true

linters-settings:
  gocyclo:
    # Minimal code complexity to report.
    # Default: 30 (but we recommend 10-20)
    min-complexity: 10
```

# nolint

有些时候因为一些特殊原因，比如某个函数无法更改，或至少在某个时段暂时达不到 linter 标准，我们希望跳过格式检查，怎么办呢？

这个时候可以使用 nolint 命令。

nolint 有两个层面的问题需要考虑：

1. 规则范围；
2. 生效范围。

前者指的是你需要排除哪些 linter 规则的影响，比如我只想针对 gocyclo 策略忽略 linter，就可以直接加上 //nolint:gocyclo 即可。

后者描述的是规则生效范围，比如是针对一行，还是一个代码块，还是一个文件。事实上 nolint 对这些层级都提供了支持。

- 行级别

```csharp
csharp复制代码var bad_name int //nolint:all

var bad_name int //nolint:golint,unused
```

注意，这里 all 代表着所有 linter 规则都忽略。你也可以指定要忽略的 linter，用逗号分隔。

- 函数/代码块级别

```go
go复制代码//nolint:all
func allIssuesInThisFunctionAreExcluded() *string {
  // ...
}

//nolint:govet
var (
  a int
  b int
)
```

- 文件级别

```go
go复制代码//nolint:unparam
package pkg
```

同时，仅仅一个 nolint 很有可能是迷惑性的，就像很多 TODO, FIXME，我们也不知道谁留的，不知道什么时候 DO,FIX。不知道什么时候让这个函数/行/文件达到 linter 要求。

所以，建议在 nolint 之后加上说明，类似这样：

```go
go复制代码//nolint:gocyclo // This legacy function is complex but the team too busy to simplify it
func someLegacyFunction() *string {
  // ...
}
```

最后一个关键点，注意我们上面的 nolint 都是紧跟着 // 注释符号的，这也是 Go 的规范，机器识别的注释不应该有空格（开发者自己看的是需要加一个空格的）。

# 参考

[1]: https://github.com/golangci/awesome-go-linters	"go-linters"
[2]: https://golangci-lint.run/usage/quick-start/	"golangci-lint 官方文档"

