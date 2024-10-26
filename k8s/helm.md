

# 简介

[Helm](https://helm.sh/zh/) 是 Kubernetes 的包管理工具，就像 Linux 下的包管理器，如：yum、apt 等，可以很方便的将之前打包好的 yaml 文件部署到 Kubernetes 上。

**Helm 解决了什么问题 **

对于复杂应用，可能会有很多资源描述文件，Helm 将其作为一个整体管理，实现资源文件的高效复用， 并对应用级别进行版本管理。

**Helm 中的三大概念**

- Chart：代表着 Helm 包。它包含在 Kubernetes 集群内部运行应用程序，工具或服务所需的所有资源定义。你可以把它看作是 Homebrew formula，Apt dpkg，或 Yum RPM 在 Kubernetes 中的等价物。

- Repository：用来存放和共享 charts 的地方。它就像 Perl 的 [CPAN 档案库网络](https://www.cpan.org/) 或是 Fedora 的 [软件包仓库](https://src.fedoraproject.org/)，只不过它是供 Kubernetes 包所使用的。

- Release：运行在 Kubernetes 集群中的 chart 的实例。一个 chart 通常可以在同一个集群中安装多次。每一次安装都会创建一个新的 release 。

# 安装

安装 Helm 先决条件：一个 Kubernetes 集群。对于 Helm 的最新版本，建议使用 Kubernetes 的最新稳定版， 在大多数情况下，它是倒数第二个次版本

> 版本具体对应关系查看 [Helm 和Kubernetes 版本对应关系](https://helm.sh/zh/docs/topics/version_skew/)

**安装步骤**

```shell
# 下载 helm
wget https://get.helm.sh/helm-v3.6.3-linux-amd64.tar.gz

# 解压 helm
tar -zxvf helm-v3.6.3-linux-amd64.tar.gz

# 移动到指定目录
mv linux-amd64/helm /usr/local/bin/helm

# 配置命令补全
helm completion bash | sudo tee /etc/bash_completion.d/helm > /dev/null
source /usr/share/bash-completion/bash_completion
```

# 常用命令

| 命令       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| create     | 创建一个 chart 并指定名字                                    |
| dependency | 管理 chart 依赖                                              |
| get        | 下载一个 release。可用的子命令：all、hooks、manifest、notes、values |
| history    | 获取 release 历史                                            |
| install    | 安装一个 chart                                               |
| list       | 列出 release                                                 |
| package    | 将 chart 目录打包到chart存档文件中                           |
| pull       | 从远程仓库中下载 chart 并解压到本地。比如：helm install [NAME] --untar |
| repo       | 添加、列出、移除、更新和索引 chart 仓库。可用的子命令：add、index、list、remove、update |
| rollback   | 从之前的版本回退                                             |
| search     | 根据关键字搜索 chart。可用的子命令：all、chart、readme、values |
| show       | 查看 chart 的详细信息。可用的子命令：all、chart、readme、values |
| status     | 显示已命名版本的状态                                         |
| template   | 本地呈现模板                                                 |
| uninstall  | 卸载一个 release                                             |
| upgrade    | 更新一个 release                                             |
| version    | 查看 Helm 客户端版本                                         |

```shell
# 添加 chart 仓库
helm repo add [NAME] [URL]

# 查看 chart 仓库列表
helm repo list

# 从 chart 仓库中更新本地可用 chart 的信息
helm repo update

# 删除 chart 仓库
helm repo remove [REPO]

# 查找 chart
helm search hub [KEY]
helm search repo [KEY]

# 查看 chart 信息
helm show all [CHART]
helm show chart [CHART]
helm show values [CHART]

# 安装 chart
helm install [NAME] [CHART]

# 查看 release 列表
helm list

# 查看 release  状态
helm status RELEASE_NAME

# 创建 chart
helm create NAME

# 安装 chart 
helm install [NAME] [CHART]

# 对 chart 进行打包
helm package [CHART_PATH]

# 查看实际模板被渲染后的文件
helm get manifest RELEASE_NAME

# 升级 chart
helm upgrade --set xxx=xxx [RELEASE] [CHART]
helm upgrade -f values.yaml [RELEASE] [CHART]

# 查看升级版本
helm history RELEASE_NAME

# 回滚
helm rollback <RELEASE> [REVISION]
# 卸载
helm uninstall RELEASE_NAME
```

# Chart 

## 文件结构

Chart 是一个组织在文件目录中的集合。目录名称就是 chart 名称（没有版本信息），因此描述 example 的 chart 可以存储在  `example /` 目录中。

```shell
example/
├── charts # 包含依赖的其他 chart
├── Chart.yaml # 用于描述这个 chart 的基本信息，包括名字、描述信息以及版本等。
├── templates # 模板目录， 当和 values 结合时，可生成有效的 Kubernetes manifest 文件
│   ├── deployment.yaml
│   ├── _helpers.tpl # 放置可以通过 chart 复用的模板辅助对象
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt # 用于介绍 chart 帮助信息， helm install 部署后展示给用户。例如：如何使用这个 chart、列出缺省的设置等。
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml # chart 默认的配置值
```

## 自定义配置

Chart 的配置选项安装过程中有两种方法可以传递配置数据： 

- `--values/-f`：指定带有覆盖的 YAML 文件。这里可以多次指定，最右边的文件优先。
- `--set`：在命令行上指定替代。如果两种都用，那么 `--set` 的优先级高。

```shell
# 先将修改的变量写到一个文件中
helm show values xxx > config.yaml

# 修改文件内容
vi config.yaml

# 使用 --values 来替换默认的配置
helm install xx xxx -f config.yaml

# 或使用 set 修改某个变量
helm install xx --set var="value" xxx
```

Value 对象来源于多个位置： 

- chart 中的  `values.yaml` 文件。

- 父 chart 中的 `values.yaml` 文件。

- 使用 `-f` 参数传递到 `helm install` 或 `helm upgrade` 的 values 文件。

- 使用 `--set` 传递的单个参数。

以上列表有明确的顺序：默认使用 `values.yaml`，可以被父 chart 的 `values.yaml` 覆盖，继而被用户提供的 values 文件覆盖，最后会被 `--set` 参数覆盖。 

## 内置对象

- 对象可以通过模板引起传递到模板中，当然我们也可以通过代码传递对象（如：`with`、`range`）。

- 对象可以非常简单仅仅有一个值，也可以包含其他对象或方法。如：Release 对象可以包含其他对象（如：`Release.Name`），File 对象有一组方法。

### Release 对象

Release 对象是我们可以在模板中访问的顶层对象之一，Release 对象描述了 版本发布本身，包含如下对象：

| 对象              | 说明                                                |
| :---------------- | --------------------------------------------------- |
| Release.Name      | release 名称                                        |
| Release.Namespace | 版本中包含的命名空间(如果 manifest 没有覆盖的话)    |
| Release.IsUpgrade | 如果当前操作是升级或回滚的话，该值将被设置为true    |
| Release.IsInstall | 如果当前操作是安装的话，该值将被设置为true          |
| Release.Revision  | 此次修订的版本号。安装时是1，每次升级或回滚都会自增 |
| Release.Service   | release 服务的名称                                  |

### Value 对象

Value 对象是从 `values.yaml` 文件和用户提供的文件传进模板的。如：`values.yaml` 中的值是 `favoriteDrink: coffee`，那么在 `template/` 目录中的模板文件中，就可以使用 `{{ .Values.favoriteDrink }}` 获取到该值。

### Chart 对象

Chart 对象是从 `Chart.yaml` 文件传进模板的。`Chart.yaml` 里面的所有数据都可以通过 Chart 获取到，如：`{{ .Chart.Name }}-{{ .Chart.Version }}`。

Chart.yaml 的内容如下：

```shell
apiVersion: chart API 版本 （必需）
name: chart 名称 （必需）
version: 语义化2 版本（必需）
kubeVersion: 兼容 Kubernetes 版本的语义化版本（可选）
description: 一句话对这个项目的描述（可选）
type: chart类型 （可选）
keywords:
  - 关于项目的一组关键字（可选）
home: 项目 home 页面的 URL （可选）
sources:
  - 项目源码的 URL 列表（可选）
dependencies: # chart 必要条件列表 （可选）
  - name: chart 名称 (nginx)
    version: chart 版本 ("1.2.3")
    repository: （可选）仓库 URL ("https://example.com/charts") 或别名 ("@repo-name")
    condition: （可选） 解析为布尔值的 yaml 路径，用于启用/禁用 chart (e.g. subchart1.enabled )
    tags: # （可选）
      - 用于一次启用/禁用 一组 chart 的 tag
    import-values: # （可选）
      - ImportValue 保存源值到导入父键的映射。每项可以是字符串或者一对子/父列表项
    alias: （可选） chart 中使用的别名。当你要多次添加相同的 chart 时会很有用
maintainers: # （可选）
  - name: 维护者名字 （每个维护者都需要）
    email: 维护者邮箱 （每个维护者可选）
    url: 维护者 URL （每个维护者可选）
icon: 用做 icon 的 SVG 或 PNG 图片 URL （可选）
appVersion: 包含的应用版本（可选）。不需要是语义化，建议使用引号
deprecated: 不被推荐的 chart （可选，布尔值）
annotations:
  example: 按名称输入的批注列表 （可选）.
```

### Files 对象

Files 在 chart 中提供访问所有的非特殊文件的对象。你不能使用它访问 Template 对象，只能访问其他文件。

- ·`Files.Ge` ：通过文件名获取文件的方法。

- `Files.GetBytes` ：用字节数组代替字符串获取文件内容的方法。 对图片之类的文件很有用。

- `Files.Glob` ：用给定的 shell glob 模式匹配文件名返回文件列表的方法。

- `Files.Lines` ：逐行读取文件内容的方法。迭代文件中每一行时很有用。

- `Files.AsSecrets`： 使用 Base 64 编码字符串返回文件体的方法。

- `Files.AsConfig` ：使用 YAML 格式返回文件体的方法。

### Capabilities 对象

Capabilities： 提供关于 Kubernetes 集群支持功能的信息。

### Template 对象

Template 包含当前被执行的当前模板信息。

> 具体对象介绍可以查看 [Helm | 内置对象](https://helm.sh/zh/docs/chart_template_guide/builtin_objects/)

## 管道和函数

### 函数

```yaml
# 调用模板指令中的 quote 函数将 .values 对象中的字符串属性用引号引起来，然后放到模板中
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ quote .Values.favorite.drink }}
  food: {{ quote .Values.favorite.food }}
```

### 管道

```yaml
# 使用管道符 | 将参数发送给函数
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  food: {{ .Values.favorite.food | quote }}
```

### default 函数

```yaml
# 指定默认值
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
```

## 流程控制

Helm 提供了三种流程控制： 

- `if/else`：用来创建条件语句。

- `with`：用来指定范围。

- `range` ：提供 `for each` 类型的循环。

除了这些之外，还提供了一些声明和使用命名模板的关键字： 

- `define`：在模板中声明一个新的命名模板。

- `template`： 导入一个命名模板。

- `block` ：声明一种特殊的可填充的模板块。

### if-else

```yaml
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if .Values.favorite.drink }} # 注意，此处向左删除空白
  mug: "true" 
  {{- end }} # 注意，此处取消的缩进，# 注意，此处向左删除空白
```

### with

```yaml
# with 允许我们为特定对象设定当前作用域 .
{{ with PIPELINE }}
  # restricted scope
{{ end }}
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink }}
  food: {{ .food }}
  {{- end }}
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
{{- toYaml . | nindent 2  }}
  {{- end }}
```

`with` 已经将 . 指向了 `.Values.favorite`，`toYaml .` 就相当于 `toYaml .Values.favorite`，将 `value.yaml` 中的 `favorite` 下面的值原模原样放到模板中，`nindent 2` 的意思是在字符串的开头添加新行，并缩进 2 个空格。

### range

```yaml
{{- range .Values.test }}
    {{ . }}
{{- end }}
```

```yaml
# values.yaml
favorite:
  drink: coffee
  food: pizza 
pizzaToppings: # 增加的内容
  - mushrooms
  - cheese
  - peppers
  - onions
  
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
{{- toYaml . | nindent 2  }}
  {{- end }}
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}   
    {{- end }}
```

## 变量

```yaml
# 解决 with 中不能使用内置对象
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- $relname := .Release.Name -}} # 提前将对象赋值给变量
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }} # 使用 $变量 的方式
  {{- end }}
```

## 模板

在 Helm 中，这类在模板中重复的地方，也可以被抽取取来，放在命名模板中（命名模板是全局的，换言之，所有的模板都可以使用）。
一个常见的命名惯例是用 chart 名称作为模板前缀：`{{ define "mychart.labels" }}`。使用特定 chart 名称作为前缀可以避免可能因为 两个不同 chart 使用了相同名称的模板而引起的冲突。

### 局部的和 _ 文件

命名以下划线 `_` 开始的文件则假定 没有 包含清单内容。这些文件不会渲染为 Kubernetes 对象定义，但在其他 chart 模板中都可用。这些文件用来存储局部和辅助对象，实际上当我们第一次创建 mychart 时，会看到一个名为 `_helpers.tpl` 的文件，这个文件是模板局部的默认位置。

### 声明和使用模板

```yaml
# define 声明模板
{{- define "MY.NAME" -}}
  # body of template here
{{- end -}}
```

```yaml
# _helpers.tpl
{{- define "mychart.labels" -}}
  labels: # 注意，这里空了2个字符
    generator: helm
    date: {{ now | htmlDate }}
{{- end -}}

# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" . }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

```yaml
# template 指令是将一个模板包含在另一个模板中的方法
# _helpers.tpl
{{- define "mychart.labels" -}}
labels: # 注意，此处没有空格
  generator: helm
  date: {{ now | htmlDate }}
{{- end -}}

# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- include  "mychart.labels" . | nindent 2 }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

### 模板内部访问文件

有时想导入的是不是模板的文件并注入其内容，而无需通过模板渲染发送内容。Helm 提供了通过 `.Files` 对象访问文件的方法。不过，在我们使用模板示例之前，有些事情需要注意： 

- 可以添加额外的文件到 chart 中。虽然这些文件会被绑定。但是要小心，由于 Kubernetes 对象的限制，Chart 必须小于 1M。

- 通常处于安全考虑，一些文件无法通过 .Files 对象访问： 无法访问 `templates/` 中的文件；无法访问使用 `.helmignore` 排除的文件
- Chart 不能保留 UNIX 模式信息，因此当文件涉及到 `.Files` 对象时，文件级权限不会影响文件的可用性。

```yaml
# config1.toml
message = Hello from config 1

# config2.toml
message = This is config 2

# config3.toml
message = This is config 3

# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $files := .Files }}
  {{- range tuple "config1.toml" "config2.toml" "config3.toml" }}
  {{ . }}: |-
        {{ $files.Get . }}
  {{- end }}
```

# 参考文档

[1]: https://helm.sh/zh/docs/chart_template_guide/builtin_objects/	"Helm 官方文档"

