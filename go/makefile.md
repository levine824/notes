# 介绍

Go 提供一个名为 go 的命令，该命令可自动下载、构建、安装和测试 Go 包和命令。

Go 提供 go 命令，官方的目的是为了不需要编写 Makefile，而是能够仅使用 Go 源代码本身中的信息来构建 Go 代码。

但是，我们在 Go 项目中也不需要完全摒弃使用 make 和 Makefile，可以使用 Makefile 的“伪目标”，简化使用 go 命令的复杂性，规范团队使用 go 命令的方式，提升个人或团队的生产力。

# make 和 Makefile

make 命令行工具可以自动判断是否需要重新编译程序，实际上 make 不仅限于程序，我们可以使用它来描述任何任务，只要其他文件发生更改，某些文件就必须从其他文件自动更新。

在使用 make 命令行工具之前，我们需要编写一个名为 Makefile 的文件，该文件描述程序中文件之前的关系，并提供用于更新每个文件的命令。也就是说 Makefile 决定 make 做什么。

关于 Makefile 本文仅介绍 Makefile 的规则(格式)，如下所示：

```makefile
target ... : prerequisites ...
<Tab>command
    ...
    ...
```

或

```makefile
target ... : prerequisites ...;command
```

阅读上面示例代码，target 是目标文件，多个目标文件之间使用空格分隔，一般只有一个目标文件，也可以是“伪目标”(某个操作的名字); prerequisites 是先决条件；command 是命令，可以在 prerequisites 后面，使用分号分隔，也可以另起一行，但是必须以开头，如果想要使用其他键，可以使用内置变量 .RECIPEPREFIX 声明。

target 目标是必须的，不可省略。prerequisites 和 command 是可选的，但是二者必须存在其一。

# Go 项目使用 Makefile

在 Go 项目中使用 Makefile，一般我们只会使用“伪目标”，我们使用 go build 构建可执行文件为例，介绍 Go 项目怎么使用 Makefile。

```makefile
build: go build -o blog
```

阅读上面示例代码，我们编写一个简单的 Makefile，定义一个“伪目标” build，命令是 go build -o blog，构建名为 blog 的可执行文件。

使用 make 命令行工具，运行“伪目标”build。

运行 make build，终端打印出 Makefile 中“伪目标” build 的命令。

如果我们不想打印出执行的命令，可以在命令前面加上 @ 符号。

在实际项目开发时，我们可能需要构建多个操作系统的可执行文件，我们再编写一个 Makefile，新增三个“伪目标”，分别是windows、linux 和 darwin。

```shell
APP=blog

build:
        @go build -o $(APP)
windows:
        @GOOS=windows go build -o $(APP)-windows
linux:
        @GOOS=linux go build -o $(APP)-linux
darwin:
        @GOOS=darwin go build -o $(APP)-darwin
```

阅读上面示例代码，我们定义一个自定义变量 APP，在命令行中使用 $(APP) 调用变量，并且 GOOS 指定操作系统，使用@开头，不再打印执行命令。

运行 make windows、make linux 和 make darwin，分别构建 windows、linux 和 drawin 操作系统的可执行文件。

运行结果如下：

```shell
.
├── Makefile
├── blog
├── blog-darwin
├── blog-linux
├── blog-windows
├── go.mod
└── main.go
```

需要注意的是，如果有文件名和“伪目标”同名，那么该“伪目标”无法使用 make 命令执行指定的 command。因为 make 发现与“伪目标”同名的文件已存在，将不会再重新构建，所以就不会运行指定的 command，为了避免出现该问题，可以使用内置目标名.PHONY声明这些“伪目标”名是“伪目标”，而不是与“伪目标”同名的文件。

```makefile
APP=blog

.PHONY: help all build windows linux darwin

help:
        @echo "usage: make <option>"
        @echo "options and effects:"
        @echo "    help   : Show help"
        @echo "    all    : Build multiple binary of this project"
        @echo "    build  : Build the binary of this project for current platform"
        @echo "    windows: Build the windows binary of this project"
        @echo "    linux  : Build the linux binary of this project"
        @echo "    darwin : Build the darwin binary of this project"
all: build windows linux darwin
build:
        @go build -o $(APP)
windows:
        @GOOS=windows go build -o $(APP)-windows
linux:
        @GOOS=linux go build -o $(APP)-linux
darwin:
        @GOOS=darwin go build -o $(APP)-darwin
```

细心的读者朋友们阅读到此处，心中可能会有一个疑问，想要知道 Makefile 中包含哪些“目标”，必须查看 Makefile 文件吗?

不必如此，我们可以在 Makefile 中编写一个“伪目标” help，用于描述 Makefile 中的“伪目标”列表和使用示例等。

Make 命令运行时，如果不指定“目标”，默认执行 Makefile 文件的第一个“目标”。一般将 help 作为 Makefile 的第一个“伪目标”，我们可以执行 make 或 make help 命令，输出使用方法。

# 案例

```makefile
# 使用 shell 脚本执行结果赋值变量
GIT_REVISION 		:= $(shell git rev-parse --verify --short HEAD 2>/dev/null)
COMPILE_TIME 		:= $(shell git log -1 --format="%ad" --date=short)

# 声明伪目标
.PHONY: help build clean setup-linux

help:
	@echo "usage  : make <option>"
	@echo "options:"
	@echo "    help         : Show help"
	@echo "    build        : Build the binary of this project for current platform"
	@echo "    clean        : Clean the workspace"
	@echo "    setup-linux  : Install the necessary software on linux"

build:
	@skaffold build
	@./build/shell/argocd_push.sh 'in ${GIT_REVISION} at ${COMPILE_TIME}'

clean:
	@./build/shell/clean.sh

setup-linux:
	@./build/shell/dev-setup-linux.sh

clone-artifact:
	@git -C artifact/argocd pull || git clone https://github.com/levine824/monorepo-argocd.git artifact/argocd
```

