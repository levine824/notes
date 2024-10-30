# 简介

Prometheus 是一款基于时序数据库的开源监控告警系统。Prometheus 的基本原理是通过 HTTP 协议周期性抓取被监控组件的状态，任意组件只要提供对应的 HTTP 接口就可以接入监控，不需要任何 SDK 或者其他的集成过程，输出被监控组件信息的 HTTP 接口被叫做 exporter 。

Promethus 有以下特点：

- 支持多维数据模型：由度量名和键值对组成的时间序列数据 ；
- 内置时间序列数据库 TSDB；
- 支持 PromQL 查询语言，可以完成非常复杂的查询和分析，对图表展示和告警非常有意义；
- 支持 HTTP 的 Pull 方式采集时间序列数据；
- 支持 PushGateway 采集瞬时任务的数据；
- 支持服务发现和静态配置两种方式发现目标；
- 支持接入 Grafana；

## 组件

普罗米修斯由多个组件组成，其中许多组件是可选的： 

- Prometheus server：主要组件，用于抓取和存储指标数据。
- Exporter：实际提供监控指标数据的服务，Prometheus server 对这些 Exporter 主动发起 HTTP 请求，抓取 指标数据并保存起来。监控不同的目标有不同的 exporter，比如监控服务器的 node-exporter , 监控 mysql 的 mysqld-erporter 等。 
- Pushgateway：被动收集、暂存监控指标数据。 通常用于被监控的对象不能和 Prometheus Server 直接通信的场景中。可以通过一个程序定期向 Pushgateway 推送指标数据，Promethes Server 会定期抓取 Pushgateway 的指标数据。 
- alertmanager：用于接收、汇总、去重、发送告警信息的服务。 

## 架构

![architesture](../assets/prometheus_1.png)

- Pushgateway：中间网关，其他程序可以主动把自己所监控对象的指标推送给 Pushgateway。

- Server discovery：Prometheus Server 通过它发现 Kubernetes 集群核心组件的指标。 
- PromQL：Prometheus 自己的查询语言，用于查询 TSDB 中的指标数据。 
- Prometheus web UI：Prometheus 的 web 页面，可以查看指标和监控对象的简易图表。 
- Grafana：通过 PromQL 查询不同维度的指标，并用丰富的图表展示。

Prometheus Server 从应用程序或者 Exporter 抓取监控指标，存放在自己管理的一个 TSDB 中。在 Prometheus Server 中会定义一些规则，如果某个监控指标数据符合了某个规则，就会触发一条告警信息，此时 Prometheus Server 会根据具体的配置，把此条告警信息推送给 Alert Manager。

# 安装

Prometheus 组件大部分是使用 GO 语言编写，部署起来没有什么依赖，可以采用二进制包部署或者 Helm 部署，此处不做详细介绍，具体可以查看[Prometheus 官方文档](https://prometheus.io/docs/introduction/overview/)。

# 配置文件

配置文件 `prometheus.yml`，主要有四部分组成：

```yaml
# 全局配置
global:
...
# 告警规则配置，从哪里加载告警规则
rule_files:
  - ...
# 告警管理器配置
alerting:
...
# 配置被监控对象
scrape_configs:
  - ...
```

## 全局配置

```yaml
global:
  scrape_interval: 30s # 将抓取间隔时间设置为每 30 秒一次。默认值为每1分钟一次。
  evaluation_interval: 5s # 规则重启评估的间隔时间设置为每隔15秒评一次。默认值为每1分钟一次
```

## 告警规则配置

```yaml
alerting:
  alertmanagers:
    - static_configs:
    - targets:
```

## 自动发现配置

```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "xxx"
    static_configs:
      - targets: ["xxx.xxx.xxx.xxx:xxxx"]
    labels:
      "<labelname>": "<labelvalue>"
```

标签 `job_name=<job name>` 会被自动添加到其下面的所有被监控对象的每个指标数据中。

标签 labels 可以配置向抓取到的每条指标数据中添加的自定义的标签。 

通过 static_configs 配置项下的 targets 中的列表配置的对象，称为静态文件的方式发现被监控对象。 

对被监控对象抓取指标时，默认使用 http 协议，URL 为 `[IP:PORT]/metrics` 。

检查配置文件语法可以使用以下命令：

```shell
./promtool check config prometheus.yml
```

# PromQL

Prometheus 提供了一种名为 PromQL 的函数式查询语言，允许用户实时选择和聚合时间序列数据。表达式的结果既可以显示为图形，也可以在 Prometheus 的表达式浏览器中作为表格数据查看，或者通过 HTTPAPI 由外部系统使用。

具体语法查看[Prometheus 官方文档](https://prometheus.io/docs/introduction/overview/)。

```shell
# 查询指定 mertic_name
node_cpu_seconds_total

# 带标签的查询
node_cpu_seconds_total{instance="<IP>:<PORT>"}

# 多标签查询
node_cpu_seconds_total{instance="<IP>:<PORT>", mode="system"}

# 计算 CPU 使用率
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance) * 100)
```

# AlertManager

Prometheus 将告警分为两个部分：Prometheus 和 Alertmanager。其中 Prometheus 配置告警触发规则，对指标进行监控和计算，将再将告警信息发送到 Alertmanager 中。Alertmanager 对告警进行管理，比如合并抑制等操作。

Alertmanager 处理由客户端应用程序发送的警报。它负责将重复数据删除，分组和路由到正确的接收者，通知方式有电子邮件、短信、微信等。它还负责沉默和禁止警报。Altermanager 有以下几个功能：

- 分组：当机房网络故障时，或者机房断电时，会突然有大量相同的告警出现，此时就需要对告警进行分组聚合，整合成少量告警发出。每个告警信息中会列出受影响的服务或者机器。

- 抑制：当集群故障时，会引发一系列告警，此时只需要通知集群故障，其它因为集群故障引起的告警应该被抑制，避免干扰判断。

- 静默：当集群升级、服务更新的过程中，大概率导致告警。因此在升级期间，对这些告警进行静默，不再发送相关告警。

## 配置

### 全局配置

```yaml
# global 指定了默认的接收者配置项				YAML		复制代码
global:
  # 默认smtp 配置项，如果recivers中没有配置则采用全局配置项 
  [ smtp_from: <tmpl_string> ]
  [ smtp_smarthost: <string> ]
  [ smtp_hello: <string> | default = "localhost" ]
  [ smtp_auth_username: <string> ]
  [ smtp_auth_password: <secret> ]
  [ smtp_auth_identity: <string> ]
  [ smtp_auth_secret: <secret> ]
  [ smtp_require_tls: <bool> | default = true ]

  # 企业微信告警配置
  [ wechat_api_url: <string> | default = "https://qyapi.weixin.qq.com/cgi-bin/" ]
  [ wechat_api_secret: <secret> ]
  [ wechat_api_corp_id: <string> ]

  # 默认http客户端配置，不推荐配置。参考官方文档: https://prometheus.io/docs/alerting/latest/clients/
  [ http_config: <http_config> ]

  # 如果警报不包含EndsAt，则ResolveTimeout是Alertmanager使用的默认值，经过此时间后，如果尚未更新，则可以将警报声明为已解决。
  # 这对Prometheus的警报没有影响，因为它们始终包含EndsAt。
  [ resolve_timeout: <duration> | default = 5m ]

# 定义通知模板，最好一个列表元素可以使用Linux通配符，如 *.tmpl
templates:
  [ - <filepath> ... ]

# 定义路由
route: <route>

# 定义通知的接收者
receivers:
  - <receiver> ...

# 告警抑制规则
inhibit_rules:
  [ - <inhibit_rule> ... ]
```

### 路由配置

每个警报都会在已配置的顶级路由处进入路由树，该路由树必须与所有警报匹配（即没有任何已配置的匹配器）。然后遍历子节点。如果将 continue 设置为 false，它将在第一个匹配的子项之后停止。如果在匹配的节点上为 true，则警报将继续与后续的同级进行匹配。如果警报与节点的任何子节点都不匹配（不匹配的子节点或不存在子节点），则根据当前节点的配置参数来处理警报。

```yaml
# 告警接收者
[ receiver: <string> ]
# 告警根据标签进行分组，相同标签的作为一组进行聚合，发送单条告警信息。特殊值 '...' 表示告警不聚合
[ group_by: '[' <labelname>, ... ']' ]

# 告警是否匹后续的同级节点，如果为true还会继续进行规则匹配，否则匹配成功就截止
[ continue: <boolean> | default = false ]

# 报警必须匹配到labelname，否则无法匹配到该组路由，一般用于发送给不同联系人时使用
match:
  [ <labelname>: <labelvalue>, ... ]
match_re:
  [ <labelname>: <regex>, ... ]

# 第一次发送当前group报警等待的时间，目的是实现同组告警的聚合
[ group_wait: <duration> | default = 30s ]

# 当上一次group告警发送成功后，改组又出现新的告警，那么等待多久再发送，一般设置为5分钟或者更久
[ group_interval: <duration> | default = 5m ]

# 已经发送成功的告警，但是一直没消除，那么等待多久再发送。一般推荐三个小时以上
[ repeat_interval: <duration> | default = 4h ]

# 子路由
routes:
  [ - <route> ... ]
```

### 接收者配置

```yaml
# 接收者名称，唯一标识
name: <string>

# 接收方式
email_configs:
  [ - <email_config>, ... ]
pagerduty_configs:
  [ - <pagerduty_config>, ... ]
pushover_configs:
  [ - <pushover_config>, ... ]
slack_configs:
  [ - <slack_config>, ... ]
opsgenie_configs:
  [ - <opsgenie_config>, ... ]
webhook_configs:
  [ - <webhook_config>, ... ]
victorops_configs:
  [ - <victorops_config>, ... ]
wechat_configs:
  [ - <wechat_config>, ... ]
```

# 数据存储

## 本地存储

```shell
/usr/local/prometheus/data/		# 数据存储目录
├── 01FK01XFQDAQ3C9APNKEFT18ZT	# 数据块
│   ├── chunks					# 数据样本
│   │   └── 000001
│   ├── index					# 索引文件
│   ├── meta.json				# 元数据文件
│   └── tombstones				# 逻辑数据
├── 01FM3SZ21W30R9N0W50KM4HKDA
│   ├── chunks
│   │   └── 000001
│   ├── index
│   ├── meta.json
│   └── tombstones
├── chunks_head
├── lock
├── queries.active
└── wal						# 预写日志文件，即原始时间序列数据，还未压缩为块存储到本地磁盘
    ├── 00000086
    ├── 00000087
    ├── 00000088
    └── checkpoint.00000085
        └── 00000000
```

## 远程存储

尽管数据是不会占用过多的磁盘空间的，但 Prometheus 依然提供了远程存储的方式。

修改 prometheus.yml 指定远程存储：

```shell
# 远程写入地址
remote_write:
  - url: "http://192.168.153.205:8086/api/v1/prom/write?db=prometheus"
# 远程读取地址
remote_read:
  - url: "http://192.168.153.205:8086/api/v1/prom/read?db=prometheus"
```

