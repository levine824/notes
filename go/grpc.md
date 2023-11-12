# 简介

gRPC 是一个高性能、开源、通用的 RPC 框架，由 Google 推出，基于 HTTP2 协议标准设计开发，默认采用 Protocol Buffers 数据序列化协议，支持多种开发语言。gRPC 提供了一种简单的方法来精确的定义服务，并且为客户端和服务端自动生成可靠的功能库。

在 gRPC 客户端可以直接调用不同服务器上的远程程序，使用姿势看起来就像调用本地程序一样，很容易去构建分布式应用和服务。和很多 RPC 系统一样，服务端负责实现定义好的接口并处理客户端的请求，客户端根据接口描述直接调用需要的服务。客户端和服务端可以分别使用 gRPC 支持的不同语言实现。

**特性**：

- 强大的 IDL

  gRPC 使用 ProtoBuf 来定义服务，ProtoBuf 是由 Google 开发的一种数据序列化协议（类似于 XML、JSON、hessian）。ProtoBuf能够将数据进行序列化，并广泛应用在数据存储、通信协议等方面；

- 多语言支持

  gRPC 支持多种语言，并能够基于语言自动生成客户端和服务端功能库。目前已提供了 C 版本 grpc、Java 版本 grpc-java 和 Go 版本grpc-go，其它语言的版本正在积极开发中，其中，grpc 支持 C、C++、Node.js、Python、Ruby、Objective-C、PHP和C# 等语言，grpc-java 已经支持 Android 开发；

- HTTP2

  gRPC 基于 HTTP2 标准设计，所以相对于其他 RPC 框架，gRPC 带来了更多强大功能，如双向流、头部压缩、多复用请求等。这些功能给移动设备带来重大益处，如节省带宽、降低TCP链接次数、节省CPU使用和延长电池寿命等。同时，gRPC 还能够提高了云端服务和 Web 应用的性能。gRPC 既能够在客户端应用，也能够在服务器端应用，从而以透明的方式实现客户端和服务器端的通信和简化通信系统的构建。



# 安装

```shell
# 下载 protobuf
go get github.com/golang/protobuf/proto
# 下载 gRPC
go get google.golang.org/grpc
```

如果无法访问 golang.org，可用如下命令代替：

```shell
git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
git clone https://github.com/golang/net.git $GOPATH/src/golang.org/x/net
git clone https://github.com/golang/text.git $GOPATH/src/golang.org/x/text
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
git clone https://github.com/google/go-genproto.git $GOPATH/src/google.golang.org/genproto
cd $GOPATH/src/
go install google.golang.org/grpc
```

```shell
# 安装 protoc-gen-go.exe，会在 GOPATH/bin 下生成 protoc-gen-go.exe
go install github.com/golang/protobuf/protoc-gen-go@latest
# 安装 protoc.exe，由于 windows 平台编译受限，很难自己手动编译，直接去网站下载一个
# 地址：https://github.com/protocolbuffers/protobuf/releases/tag/v3.9.0 ，同样放在 GOPATH/bin 下
```



# ProtoBuf

Protobuf 是 Google 给出的一种通用的数据表示方式，通过 proto 文件定义的数据格式，可以一键式的生成 C++，Python，Java 等各种语言实现。Protobuf 经历了 protobuf2 和 protobuf3，pb3 比 pb2 简化了很多，目前主流的版本是 pb3。

![grpc_1](..\assets\grpc_1.png)

## 基本规范

- 文件以 .proto 做为文件后缀，除结构定义外的语句以分号结尾；

- 结构定义可以包含：message、service、enum；

- rpc 方法定义结尾的分号可有可无；

- Message 命名采用驼峰命名方式，字段命名采用小写字母加下划线分隔方式：

  ```
    message SongServerRequest {
        required string song_name = 1;
    }
  ```

- Enums 类型名采用驼峰命名方式，字段命名采用大写字母加下划线分隔方式：

  ```
    enum Foo {
        FIRST_VALUE = 1;
        SECOND_VALUE = 2;
    }
  ```

- Service 与 rpc 方法名统一采用驼峰式命名。

## 字段规则

- 字段格式：`限定修饰符 | 数据类型 | 字段名称 | = | 字段编码值 | [字段默认值]`；
- 限定修饰符包含 required\optional\repeated
  - Required: 表示是一个必须字段，必须相对于发送方，在发送消息之前必须设置该字段的值，对于接收方，必须能够识别该字段的意思。发送之前没有设置 required 字段或者无法识别 required 字段都会引发编解码异常，导致消息被丢弃；
  - Optional：表示是一个可选字段，可选对于发送方，在发送消息时，可以有选择性的设置或者不设置该字段的值。对于接收方，如果能够识别可选字段就进行相应的处理，如果无法识别，则忽略该字段，消息中的其它字段正常处理。因为 optional 字段的特性，很多接口在升级版本中都把后来添加的字段都统一的设置为 optional 字段，这样老的版本无需升级程序也可以正常的与新的软件进行通信，只不过新的字段无法识别而已，因为并不是每个节点都需要新的功能，因此可以做到按需升级和平滑过渡；
  - Repeated：表示该字段可以包含 0~N 个元素。其特性和optional一样，但是每一次可以包含多个值。可以看作是在传递一个数组的值。
- 数据类型
  - Protobuf 定义了一套基本数据类型。几乎都可以映射到 C++\Java 等语言的基础数据类型：

| .proto   | C++    | Java       | Python         | Go      | Ruby                 | C#         |
| :------- | :----- | :--------- | :------------- | :------ | :------------------- | :--------- |
| double   | double | double     | float          | float64 | Float                | double     |
| float    | float  | float      | float          | float32 | Float                | float      |
| int32    | int32  | int        | int            | int32   | Fixnum or Bignum     | int        |
| int64    | int64  | long       | ing/long[3]    | int64   | Bignum               | long       |
| uint32   | uint32 | int[1]     | int/long[3]    | uint32  | Fixnum or Bignum     | uint       |
| uint64   | uint64 | long[1]    | int/long[3]    | uint64  | Bignum               | ulong      |
| sint32   | int32  | int        | intj           | int32   | Fixnum or Bignum     | int        |
| sint64   | int64  | long       | int/long[3]    | int64   | Bignum               | long       |
| fixed32  | uint32 | int[1]     | int            | uint32  | Fixnum or Bignum     | uint       |
| fixed64  | uint64 | long[1]    | int/long[3]    | uint64  | Bignum               | ulong      |
| sfixed32 | int32  | int        | int            | int32   | Fixnum or Bignum     | int        |
| sfixed64 | int64  | long       | int/long[3]    | int64   | Bignum               | long       |
| bool     | bool   | boolean    | boolean        | bool    | TrueClass/FalseClass | bool       |
| string   | string | String     | str/unicode[4] | string  | String(UTF-8)        | string     |
| bytes    | string | ByteString | str            | []byte  | String(ASCII-8BIT)   | ByteString |

```
+ N 表示打包的字节并不是固定。而是根据数据的大小或者长度
+ 关于 fixed32 和 int32 的区别。fixed32 的打包效率比 int32 的效率高，但是使用的空间一般比 int32 多。因此一个属于时间效率高，一个属于空间效率高
```

- 字段名称
  - 字段名称的命名与 C、C++、Java 等语言的变量命名方式几乎是相同的；
  - protobuf 建议字段的命名采用以下划线分割的驼峰式。例如：first_name 而不是 firstName；
- 字段编码值
  - 有了该值，通信双方才能互相识别对方的字段，相同的编码值，其限定修饰符和数据类型必须相同，编码值的取值范围为 `1~2^32`（4294967296）；
  - 其中 1~15 的编码时间和空间效率都是最高的，编码值越大，其编码的时间和空间效率就越低，所以建议把经常要传递的值把其字段编码设置为1-15 之间的值；
  - 1900~2000 编码值为 Google protobuf 系统内部保留值，建议不要在自己的项目中使用；
- 字段默认值
  - 当在传递数据时，对于 required 数据类型，如果用户没有设置值，则使用默认值传递到对端。

## Service 如何定义

- 如果想要将消息类型用在 RPC 系统中，可以在 .proto 文件中定义一个 RPC 服务接口，protocol buffer 编译器会根据所选择的不同语言生成服务接口代码；
- 例如，想要定义一个 RPC 服务并具有一个方法，该方法接收 SearchRequest 并返回一个 SearchResponse，此时可以在 .proto 文件中进行如下定义：

```
    service SearchService {
        rpc Search (SearchRequest) returns (SearchResponse) {}
    }
```

- 生成的接口代码作为客户端与服务端的约定，服务端必须实现定义的所有接口方法，客户端直接调用同名方法向服务端发起请求，比较麻烦的是，即便业务上不需要参数也必须指定一个请求消息，一般会定义一个空 message。

## Message 如何定义

- 一个 message 类型定义描述了一个请求或响应的消息格式，可以包含多种类型字段；
- 例如：定义一个搜索请求的消息格式，每个请求包含查询字符串、页码、每页数目；
- 字段名用小写，转为 go 文件后自动变为大写，message 就相当于结构体：

```
    syntax = "proto3";

    message SearchRequest {
        string query = 1;            // 查询字符串
        int32  page_number = 2;     // 页码
        int32  result_per_page = 3;   // 每页条数
    }
```

- 首行声明使用的 protobuf 版本为 proto3；
- SearchRequest 定义了三个字段，每个字段声明以分号结尾，.proto 文件支持双斜线 // 添加单行注释。

## 添加更多 Message 类型

- 一个 .proto 文件中可以定义多个消息类型，一般用于同时定义多个相关的消息，例如在同一个 .proto 文件中同时定义搜索请求和响应消息：

```
    syntax = "proto3";

    // SearchRequest 搜索请求
    message SearchRequest {
        string query = 1;            // 查询字符串
        int32  page_number = 2;     // 页码
        int32  result_per_page = 3;   // 每页条数
    }

    // SearchResponse 搜索响应
    message SearchResponse {
        ...
    }
```

## 如何使用其他 Message

- message 支持嵌套使用，作为另一 message 中的字段类型：

```
    message SearchResponse {
        repeated Result results = 1;
    }

    message Result {
        string url = 1;
        string title = 2;
        repeated string snippets = 3;
    }
```

## Message 嵌套的使用

- 支持嵌套消息，消息可以包含另一个消息作为其字段。也可以在消息内定义一个新的消息；
- 内部声明的 message 类型名称只可在内部直接使用：

```
    message SearchResponse {
        message Result {
            string url = 1;
            string title = 2;
            repeated string snippets = 3;
        }
        repeated Result results = 1;
    }
```

- 另外，还可以多层嵌套：

```
    message Outer {                // Level 0
        message MiddleAA {        // Level 1
            message Inner {        // Level 2
                int64 ival = 1;
                bool  booly = 2;
            }
        }
        message MiddleBB {         // Level 1
            message Inner {         // Level 2
                int32 ival = 1;
                bool  booly = 2;
            }
        }
    }
```

## proto3 的 Map 类型

- proto3 支持 map 类型声明：

```
    map<key_type, value_type> map_field = N;

    message Project {...}
    map<string, Project> projects = 1;
```

- 键、值类型可以是内置的类型，也可以是自定义 message 类型；
- 字段不支持 repeated 属性。

## .proto 文件编译

- 通过定义好的 .proto 文件生成 Java, Python, C++, Go, Ruby, JavaNano, Objective-C, or C# 代码，需要安装编译器 protoc；
- 当使用 protocol buffer 编译器运行 .proto 文件时，编译器将生成所选语言的代码，用于使用在 .proto 文件中定义的消息类型、服务接口约定等。不同语言生成的代码格式不同：
  - C++：每个 .proto 文件生成一个 .h 文件和一个 .cc 文件，每个消息类型对应一个类；
  - Java：生成一个 .java 文件，同样每个消息对应一个类，同时还有一个特殊的 Builder 类用于创建消息接口；
  - Python：姿势不太一样，每个 .proto 文件中的消息类型生成一个含有静态描述符的模块，该模块与一个元类 metaclass 在运行时创建需要的 Python 数据访问类；
  - Go：生成一个 .pb.go 文件，每个消息类型对应一个结构体；
  - Ruby： 生成一个 .rb 文件的 Ruby 模块，包含所有消息类型；
  - JavaNano：类似 Java，但不包含 Builder 类；
  - Objective-C：每个 .proto 文件生成一个 pbobjc.h 和一个 pbobjc.m 文件；
  - C#：生成 .cs 文件包含，每个消息类型对应一个类。

## import 导入定义

- 可以使用 import 语句导入使用其它描述文件中声明的类型；
- protobuf 接口文件可以像 C 语言的 h 文件一个，分离为多个，在需要的时候通过 import 导入需要对文件。其行为和 C 语言的#include 或者 java 的 import 的行为大致相同，例如：import “others.proto”;
- protocol buffer 编译器会在 -I / –proto_path 参数指定的目录中查找导入的文件，如果没有指定该参数，默认在当前目录中查找。

## 包的使用

- 在 .proto 文件中使用 package 声明包名，避免命名冲突：

```
syntax = "proto3";
package foo.bar;
message Open {...}
```

- 在其他的消息格式定义中可以使用包名+消息名的方式来使用类型，如：

```
message Foo {
    ...
    foo.bar.Open open = 1;
    ...
}
```

- 在不同的语言中，包名定义对编译后生成的代码的影响不同
  - C++ 中：对应 C++ 命名空间，例如：Open 会在命名空间 foo::bar 中；
  - Java 中：package 会作为 Java 包名，除非指定了 option jave_package 选项；
  - Python 中：package 被忽略；
  - Go 中：默认使用 package 名作为包名，除非指定了 option go_package 选项；
  - JavaNano 中：同 Java；
  - C# 中：package 会转换为驼峰式命名空间，如：Foo.Bar,除非指定了 option csharp_namespace 选项。

# Protoc 的使用

## 安装

```shell
# 下载 proto 和 grpc
go get github.com/golang/protobuf/proto
go get google.golang.org/grpc
# 安装 protoc-gen-go 
go install github.com/golang/protobuf/protoc-gen-go@latest
go install github.com/golang/protobuf/protoc-gen-go-grpc@latest
```

## 编写 proto 文件

```go
syntax = "proto3"; // 指定proto版本
package hello;     // 指定默认包名

// 指定golang包名
option go_package = "hello";

// 定义Hello服务
service Hello {
    // 定义SayHello方法
    rpc SayHello(HelloRequest) returns (HelloResponse) {}
}

// HelloRequest 请求结构
message HelloRequest {
    string name = 1;
}

// HelloResponse 响应结构
message HelloResponse {
    string message = 1;
}
```

## 生成 go 文件

```shell
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./auth.proto
```

## 实现服务端接口

```go
syntax = "proto3"; // 指定proto版本
package hello;     // 指定默认包名

// 指定golang包名
option go_package = "hello";

// 定义Hello服务
service Hello {
    // 定义SayHello方法
    rpc SayHello(HelloRequest) returns (HelloResponse) {}
}

// HelloRequest 请求结构
message HelloRequest {
    string name = 1;
}

// HelloResponse 响应结构
message HelloResponse {
    string message = 1;
}
```

## 客户端实现

```go
package main

import (
    pb "github.com/jergoo/go-grpc-example/proto/hello" // 引入proto包
    "golang.org/x/net/context"
    "google.golang.org/grpc"
    "google.golang.org/grpc/grpclog"
)

const (
    // Address gRPC服务地址
    Address = "127.0.0.1:50052"
)

func main() {
    // 连接
    conn, err := grpc.Dial(Address, grpc.WithInsecure())
    if err != nil {
        grpclog.Fatalln(err)
    }
    defer conn.Close()

    // 初始化客户端
    c := pb.NewHelloClient(conn)

    // 调用方法
    req := &pb.HelloRequest{Name: "gRPC"}
    res, err := c.SayHello(context.Background(), req)

    if err != nil {
        grpclog.Fatalln(err)
    }

    grpclog.Println(res.Message)
}
```



# buf

buf 对于 protoc 文件来说，相当于 go module 相对于 go 的作用，能够帮助我们管理 protoc 相关包和依赖。

## 安装

```go
go install github.com/bufbuild/buf/cmd/buf@v1.19.0
# 或者直接下载压缩包
```

## 示例

### 项目目录结构

```
|—— gen/ #生成后文件目录
    |—— api/
        |—— hello
           |—— v1/
               |—— hello.pb.go
               |—— hello.pb.gw.go
               |—— hello.swagger.json
               |—— hello_grpc.pb.go
|—— proto/
    |—— api/
        |—— v1/
            |—— hello.proto   // proto 描述文件
```

### buf.yaml 文件

基本配置文件，放在 proto 目录下。

```yaml
version: v1
deps: # 添加 googleapis 依赖
  - buf.build/googleapis/googleapis
lint:
  use:
    - DEFAULT
breaking:
  use:
    - FILE
```

### buf.work.yaml 文件

```yaml
version: v1
directories:
  - proto # 扫描哪些目录下的 proto 文件
```

### buf.gen.yaml 文件

```yaml
version: v1
managed:
  enabled: true
  go_package_prefix:
    # <module_name>   : name in go.mod
    # <relative_path> : where generated code should be output
    default: github.com/levine824/monorepo/
    # Remove `except` field if googleapis is not used
    except:
      - buf.build/googleapis/googleapis
plugins: # 插件本地没有的话会远程下载
  - plugin: buf.build/grpc-ecosystem/gateway:v2.15.2
    out: gen
    opt:
      - paths=source_relative
  # dependencies
  - plugin: buf.build/protocolbuffers/go
    out: gen
    opt:
      - paths=source_relative
  - plugin: buf.build/grpc/go
    out: gen
    opt:
      - paths=source_relative
  - plugin: buf.build/grpc-ecosystem/openapiv2:v2.15.2
    out: gen
```

### proto 文件

```go
syntax = "proto3";
package hello.v1;

import "google/api/annotations.proto";

service Hello{
  rpc SayHello(HelloRequest) returns(HelloReply){
    option (google.api.http) = { // 对应 http 请求
      get: "/api/v1/hello/{name}"
    };
  };
}

message HelloRequest{
  string name = 1;
}

message HelloReply{
  string message = 1;
}
```

# 认证

（具体参考链接[7]）

# 拦截器

（具体参考链接[8]）

# Http 网关

（具体参考链接[9]）

# F&Q

1. 当使用单独的文件描述 http 接口时，不要使用 remote 插件，不然无法找到。

```yaml
plugins:
  - plugin: gateway
    out: api/gen/go
    opt:
      - paths=source_relative
      - grpc_api_configuration=pong.yaml
      - standalone=true
```

# 参考

[1]: https://grpc.io/	"grpc 官方文档"
[2]: https://www.topgoer.cn/docs/golang/chapter16-7	"grpc 介绍"
[3]: https://buf.build/docs	"buf 官方文档"
[4]: https://github.com/googleapis/googleapis	"googleapis 地址"
[5]: https://grpc-ecosystem.github.io/grpc-gateway/docs/tutorials/adding_annotations/	"grpc-gateway"
[6]: https://buf.build/plugins	"buf 插件仓库"
[7]: https://www.topgoer.cn/docs/golang/chapter16-7-7	"认证"
[8]: https://www.topgoer.cn/docs/golang/chapter16-7-8	"拦截器"
[9]: https://www.topgoer.cn/docs/golang/chapter16-7-10	"网关"