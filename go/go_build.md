# go build

## 使用

`go build [-o 输出名] [-i] [编译标记] [包名]`

- 如果参数为 XX.go 文件或文件列表，则编译为一个个单独的包；
- 当编译单个main包（文件），则生成可执行文件；
- 当编译单个或多个包非主包时，只构建编译包，但丢弃生成的对象（.a），仅用作检查包可以构建；
- 当编译包时，会自动忽略 _test.go 的测试文件。

### 基本使用

go build 的使用比较简洁,所有的参数都可以忽略,直到只有 go build，这个时候意味着使用当前目录进行编译，下面的几条命令是等价的. 都是使用当前目录编译的意思。因为我们忽略了 packages，所以自然就使用当前目录进行编译了。

```shell
go build
go build .
go build hello.go
```

从这里我们也可以推测出，go build 本质上需要的是一个路径,让编译器可以找到哪些需要编译的 go 文件。packages 其实是一个相对路径，是相对于我们定义的 GOROOT 和 GOPATH 这两个环境变量的，所以有了 packages 这个参数后, go build 就可以知道哪些需要编译的 go 文件了。

### 指定包的方式

```shell
go build github.com/ourlang/noutil
go build github.com/ourlang/noutil/...
```

## 常用参数

```shell
-o
   output 指定编译输出的名称，代替默认的包名
-i
   install 安装作为目标的依赖关系的包(用于增量编译提速)
-a
    完全编译，不理会 -i 产生的 .a 文件(文件会比不带 -a 的编译出来要大？)
-n
    仅打印输出 build 需要的命令，不执行 build 动作（少用）
-p n
    开多少核 cpu 来并行编译，默认为本机 CPU 核数（少用）
-race
    同时检测数据竞争状态，只支持 linux/amd64, freebsd/amd64, darwin/amd64 和 windows/amd64
-msan
    启用与内存消毒器的互操作。仅支持 linux/amd64，并且只用 Clang/LLVM 作为主机 C 编译器（少用）
-v
    打印出被编译的包名（少用）
-work
    打印临时工作目录的名称，并在退出时不删除它（少用）
-x
    同时打印输出执行的命令名（-n）（少用）
-asmflags 'flag list'
    传递每个 go 工具 asm 调用的参数（少用）
-buildmode mode
    编译模式（少用）
    'go help buildmode'
-compiler name
    使用的编译器 == runtime.Compiler
    (gccgo or gc)（少用）
-gccgoflags 'arg list'
    gccgo 编译/链接器参数（少用）
-gcflags 'arg list'
    垃圾回收参数（少用）
-ldflags 'flag list'
    '-s -w': 压缩编译后的体积
    -s: 去掉符号表
    -w: 去掉调试信息，不能 gdb 调试了
-linkshared
    链接到以前使用创建的共享库
    -buildmode=shared
-pkgdir dir
    从指定位置，而不是通常的位置安装和加载所有软件包。例如，当使用非标准配置构建时，使用 -pkgdir 将生成的包保留在单独的位置
-tags 'tag list'
    构建出带 tag 的版本
```

## 跨平台编译

Go 提供了编译链工具，可以让我们在任何一个开发平台上，编译出其他平台的可执行文件。 默认情况下，都是根据我们当前的机器生成的可执行文件，比如你的是 Linux 64位，就会生成Linux 64位下的可执行文件，可以使用 go env 查看编译环境,以下截取重要的部分。

编译跨平台的只需要修改 GOOS、GOARCH、CGO_ENABLED 三个环境变量即可：

- GOOS：目标平台的操作系统（darwin、freebsd、linux、windows）；
- GOARCH：目标平台的体系架构 32 位还是 64 位（386、amd64、arm）；
- 交叉编译不支持 CGO 所以要禁用它。

**Window 环境下编译 Mac 和 Linux 64位可执行程序**

```bat
# Mac
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build main.go
# Linux 
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build main.go
```

**Mac 下编译 Linux 和 Windows 64位可执行程序**

```shell
# Mac
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
# Windows 
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
```

**Linux 下编译 Mac 和 Windows 64位可执行程序**

```shell
# Mac
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build main.go
# Windows
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
```



**参考链接：**

[1]: https://www.lsdcloud.com/go/middleware/go-build.html#_1-%E4%BD%BF%E7%94%A8	"go build 命令详解"

