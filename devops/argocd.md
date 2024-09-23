

# 简介

Argo CD 是以 Kubernetes 作为基础设施，遵循声明式 GitOps 理念的持续交付（continuous delivery, CD）工具，支持多种配置管理工具，包括 ksonnet/jsonnet、kustomize 和 Helm 等。它的配置和使用非常简单，并且自带一个简单易用的可视化界面。

按照官方定义，Argo CD 被实现为一个 Kubernetes 控制器，它会持续监控正在运行的应用，并将当前的实际状态与 Git 仓库中声明的期望状态进行比较，如果实际状态不符合期望状态，就会更新应用的实际状态以匹配期望状态。

# 优势

**Git 作为应用的唯一真实来源**

所有 K8s 的声明式配置都保存在 Git 中，并把 Git 作为应用的唯一事实来源，我们不再需要手动更新应用（比如执行脚本，执行 kubectl apply 或者 helm install 命令），只需要通过统一的接口（Git）来更新应用。

此外，Argo CD 不仅会监控 Git 仓库中声明的期望状态，还会监控集群中应用的实际状态，并将两种状态进行对比，只要实际状态不符合期望状态，实际状态就会被修正与期望状态一致。所以即使有人修改了集群中应用的状态（比如修改了副本数量），Argo CD 还是会将其恢复到之前的状态。这就真正确保了 Git 仓库中的编排文件可以作为集群状态的唯一真实来源。

**快速回滚**

Argo CD 会定期拉取最新配置并应用到集群中，一旦最新的配置导致应用出现了故障（比如应用启动失败），我们可以通过 Git History 将应用状态快速恢复到上一个可用的状态。

如果你有多个 Kubernetes 集群使用同一个 Git 仓库，这个优势会更明显，因为你不需要分别在不同的集群中通过 kubectl delete 或者 helm uninstall 等手动方式进行回滚，只需要将 Git 仓库回滚到上一个可用的版本，Argo CD 便会自动同步。

**集群灾备**

如果你的集群出现故障，且短期内不可恢复，可以直接创建一个新集群，然后将 Argo CD 连接到 Git 仓库，这个仓库包含了整个集群的所有配置声明。最终新集群的状态会与之前旧集群的状态一致，完全不需要人工干预。

**使用 Git 实现访问控制**

通常在生产环境中是不允许所有人访问 Kubernetes 集群的，如果直接在 Kubernetes 集群中控制访问权限，必须要使用复杂的 RBAC 规则。在 Git 仓库中控制权限就比较简单了，例如所有人（DevOps 团队，运维团队，研发团队，等等）都可以向仓库中提交 Pull Request，但只有高级工程师可以合并 Pull Request。

这样做的好处是，除了集群管理员和少数人员之外，其他人不再需要直接访问 Kubernetes 集群，只需访问 Git 仓库即可。对于程序而言也是如此，类似于 Jenkins 这样的 CI 工具也不再需要访问 Kubernetes 的权限，因为只有 Argo CD 才可以 apply 配置清单，而且 Argo CD 已经部署在 Kubernetes 集群中，必要的访问权限已经配置妥当，这样就不需要给集群外的任意人或工具提供访问的证书，可以提供更强大的安全保障。

# 架构

![argocd_1](../assets/argocd_1.png)

从功能架构来看，Argo CD 主要有三个组件：API Server、Repository Server 和 Application Controller。从 GitOps 工作流的角度来看，总共分为 3 个阶段：检索、调谐和呈现。

- 检索 -- Repository Server

检索阶段会克隆应用声明式配置清单所在的 Git 仓库，并将其缓存到本地存储。包含 Kubernetes 原生的配置清单、Helm Chart 以及 Kustomize 配置清单。履行这些职责的组件就是 Repository Server。

- 调谐 -- Application Controller

调谐（Reconcile）阶段是最复杂的，这个阶段会将 Repository Server 获得的配置清单与反映集群当前状态的实时配置清单进行对比，一旦检测到应用处于 OutOfSync 状态，Application Controller 就会采取修正措施，使集群的实际状态与期望状态保持一致。

- 呈现 -- API Server

最后一个阶段是呈现阶段，由 Argo CD 的 API Server 负责，它本质上是一个 gRPC/REST Server，提供了一个无状态的可视化界面，用于展示调谐阶段的结果。同时还提供了以下这些功能：

- 应用管理和状态报告；
- 调用与应用相关的操作（例如同步、回滚、以及用户自定义的操作）；
- Git 仓库与集群凭证管理（以 Kubernetes Secret 的形式存储）；
- 为外部身份验证组件提供身份验证和授权委托；
- RBAC 增强；
- Git Webhook 事件的监听器/转发器。

# 部署

Argo CD 有两种不同的部署模式，常用的部署模式是多租户，一般如果组织内部包含多个应用研发团队，就会采用这种部署模式。用户可以使用可视化界面或者 argocd CLI 来访问 Argo CD。argocd CLI 必须先通过 `argocd login <server-host>` 来获取 Argo CD 的访问授权。

```shell
argocd login SERVER [flags]

## Login to Argo CD using a username and password
argocd login cd.argoproj.io

## Login to Argo CD using SSO
argocd login cd.argoproj.io --sso

## Configure direct access using Kubernetes API server
argocd login cd.argoproj.io --core
```

Argo CD 有两种不同的部署模式：

## 非高可用

推荐用于测试和演示环境，不推荐在生产环境下使用。有两种部署清单可供选择：

- install.yaml：标准的 Argo CD 部署清单，拥有集群管理员权限。可以使用 Argo CD 在其运行的集群内部署应用程序，也可以通过接入外部集群的凭证将应用部署到外部集群中；

- namespace-install.yaml：这个部署清单只需要 namespace 级别的权限。如果你不需要在 Argo CD 运行的集群中部署应用，只需通过接入外部集群的凭证将应用部署到外部集群中，推荐使用此部署清单。还有一种花式玩法，你可以为每个团队分别部署单独的 Argo CD 实例，但是每个 Argo CD 实例都可以使用特殊的凭证（如`argocd cluster add <CONTEXT> --in-cluster --namespace <YOUR NAMESPACE>`）将应用部署到同一个集群中（即 `kubernetes.svc.default`，也就是内部集群）。

> 注：namespace-install.yaml 配置清单中并不包含 Argo CD 的 CRD，需要自己提前单独部署：`kubectl apply -k https://github.com/argoproj/argo-cd/manifests/crds\?ref\=stable`。

## 高可用

与非高可用部署清单包含的组件相同，但增强了高可用能力和弹性能力，推荐在生产环境中使用。

- ha/install.yaml：与上文提到的 install.yaml 的内容相同，但配置了相关组件的多个副本；
- ha/namespace-install.yaml ：与上文提到的 namespace-install.yaml 相同，但配置了相关组件的多个副本。

## Core

Core 模式也就是最精简的部署模式，不包含 API Server 和可视化界面，只部署了每个组件的轻量级（非高可用）版本。

用户需要 Kubernetes 访问权限来管理 Argo CD，因此必须使用下面的命令来配置 argocd CLI：

```shell
kubectl config set-context --current --namespace=argocd # change current kube context to argocd namespace
argocd login --core
```

### **Kustomize**

Argo CD 配置清单也可以使用 Kustomize 来部署，建议通过远程的 URL 来调用配置清单，使用 patch 来配置自定义选项。

```shell
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: argocd
resources:
- https://raw.githubusercontent.com/argoproj/argo-cd/v2.0.4/manifests/ha/install.yaml
```

### **Helm**

Argo CD 的 Helm Chart 目前由社区维护（具体参考链接[2]）。

# 参考

[1]: https://argo-cd.readthedocs.io/en/stable/	"ArgoCD 官方文档"
[2]: https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd	"ArgoCD Yaml"

