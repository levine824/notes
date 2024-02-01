# k8s 网络

k8s 网络钟需要解决的通信问题包括如下几点：

- 容器间通信：同一个 pod 内的多个容器间的通信，通过 lo 即可实现；   

- pod 之间的通信： 同一节点的 pod 之间通过 cni 网桥转发数据包。( brctl show 可以查看) 不同节点的 pod 之间的通信需要网络插件支持；
- pod 和 service 通信: 通过 iptables 或 ipvs 实现通信，ipvs 取代不了 iptables，因为 ipvs 只能做负载均衡，而做不了 nat 转换；
- pod 和外网通信：iptables 的 masquerade；
- service 与集群外部客户端的通信: ingress、nodeport、loadbalancer。

## 网络策略

### 前置条件

网络策略通过网络插件来实现。要使用网络策略，你必须使用支持 NetworkPolicy 的网络解决方案。 创建一个 NetworkPolicy 资源对象而没有控制器来使它生效的话，是没有任何作用的。( Flannel 不支持 NetworkPolicy，所以使用 Flannel 网络插件是不会隔离pod的)

### 隔离和非隔离的 pod

默认情况下，pod 是非隔离的，它们接受任何来源的流量。pod 在被某 NetworkPolicy 选中时进入被隔离状态。 一旦名字空间中有 NetworkPolicy 选择了特定的 pod，该 pod 会拒绝该 NetworkPolicy 所不允许的连接。 （名字空间下其他未被 NetworkPolicy 所选择的 pod 会继续接受所有的流量）
网络策略不会冲突，它们是累积的。 如果任何一个或多个策略选择了一个 pod, 则该 pod 受限于这些策略的 入站（Ingress）/出站（Egress）规则的并集。因此评估的顺序并不会影响策略的结果。为了允许两个 pods 之间的网络数据流，源端 pod 上的出站（Egress）规则和 目标端 pod 上的入站（Ingress）规则都需要允许该流量。 如果源端的出站（Egress）规则或目标端的入站（Ingress）规则拒绝该流量， 则流量将被拒绝。

## service 和 iptables

service 的代理是 kube-proxy，kube-proxy 运行在所有节点上，它监听 apiserver 中 service 和 endpoint 的变化情况，创建路由规则以提供服务 ip 和负载均衡功能。简单理解此进程是 service 的透明代理兼负载均衡器，其核心功能是将到某个 service 的访问请求转发到后端的多个 pod 实例上，而 kube-proxy 底层又是通过 iptables 和 ipvs 实现的。

### iptables 原理

Kubernetes 从1.2版本开始，将 iptables 作为 kube-proxy 的默认模式。iptables 模式下的 kube-proxy 不再起到 proxy 的作用，其核心功能：通过 api server 的 watch 接口实时跟踪 service 与 endpoint 的变更信息，并更新对应的 iptables 规则，client 的请求流量则通过 iptables 的 NAT 机制 “直接路由” 到目标 pod。

### ipvs 原理

ipvs 在 Kubernetes1.11 中升级为 GA 稳定版。ipvs 则专门用于高性能负载均衡，并使用更高效的数据结构，允许几乎无限的规模扩张，因此被 kube-proxy 采纳为最新模式。

在 ipvs 模式下，使用 iptables 的扩展 ipset，而不是直接调用 iptables 来生成规则链。iptables 规则链是一个线性的数据结构，ipset 则引入了带索引的数据结构，因此当规则很多时，也可以很高效地查找和匹配。

可以将 ipset 简单理解为一个 ip 段的集合，这个集合的内容可以是 ip 地址、ip 网段、端口等，iptables 可以直接添加规则对这个“可变的集合”进行操作，这样做的好处在于可以大大减少 iptables 规则的数量，从而减少性能损耗。

### ipvs 和 iptables 的异同

iptables 与 ipvs 都是基于 Netfilter 实现的，但因为定位不同，二者有着本质的差别：iptables 是为防火墙而设计的；ipvs 则专门用于高性能负载均衡，并使用更高效的数据结构，允许几乎无限的规模扩张。

与 iptables 相比，ipvs 拥有以下明显优势：

- 为大型集群提供了更好的可扩展性和性能；
- 支持比 iptables 更复杂的复制均衡算法(最小负载、最少连接、加权等)；
- 支持服务器健康检查和连接重试等功能；
- 可以动态修改 ipset 的集合，即使 iptables 的规则正在使用这个集合。

# CNI 是什么

抽象的接口层，将容器网络配置方案与容器平台方案解耦。CNI（Container Network Interface）就是这样的一个接口层，它定义了一套接口标准，提供了规范文档以及一些标准实现。采用 CNI 规范来设置容器网络的容器平台不需要关注网络的设置的细节，只需要按 CNI 规范来调用 CNI 接口即可实现网络的设置。

# CNI 插件

k8s 通过 CNI 接口接入其他插件来实现网络通讯。目前比较流行的插件有 Flannel，Calico 等。

CNI 插件存放位置：`cat /etc/cni/net.d/10-flannel.conflist `。



# 参考链接

[1]: https://blog.csdn.net/Gong_yz/article/details/129483025	"k8s 网络策略和网络插件"
[2]: https://blog.csdn.net/justlpf/article/details/129196788	"Flannel/Calico对比"
[3]: https://zhuanlan.zhihu.com/p/550373285	"深入剖析Kubernetes"

