一般在开发 kubernetes 应用的过程中，我们在本地开发调试测试完成以后，再通过 CI/CD 的方式部署到 kubernetes 集群中，这样一个完整的过程是十分耗时的，并且无法在本地运行服务，调试也将变得效率低下，因此，Skaffold 油然而生。Skaffold 是一款命令行工具，旨在促进 kubernetes 应用的持续开发，Skaffold 可以将构建、推送及向 kubernetes 集群部署应用程序的过程自动化。

# 简介

Skaffold 是由 Google 发布的命令行工具，专注于促进 K8S 应用的持续 deployment。自动化 building 和 deploying 到 kubernetes 集群的任务，可以让开发者专注于编写代码。

2019年11月份，Skaffold 普遍可用的版本发布了，承诺自动化开发工作流程，以此来节省开发者的时间。那么，Skaffold 提供哪些功能呢？

- 当你开发的时候，检测代码的变动；
- 基于你的 Dockerfile 或者 Jib 自动化 build 和创建你的 artifacts（也就是 Docker image）；
- 给 artifacts 打 tag；
- 把 artifacts 发布/部署到你的 kubernetes 集群。

**特点**

- 没有服务器端组件，所以不会增加你的集群开销；
- 自动检测源代码中的更改并自动构建/推送/部署；
- 自动更新镜像TAG，不要担心手动去更改kubernetes的 manifest 文件；
- 一次性构建/部署/上传不同的应用，因此它对于微服务同样完美适配；
- 支持开发环境和生产环境，通过仅一次运行manifest，或者持续观察变更。

# 安装

Skaffold 需要先安装 minikube/kubernetes 和 kubectl：

```shell
# 安装 minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --force --driver=docker
alias kubectl="minikube kubectl --"
# 安装 skaffold
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64
```

# 语法

```shell
skaffold [flags] [options]
End-to-end Pipelines:
  run               Run a pipeline
  dev               Run a pipeline in development mode
  debug             Run a pipeline in debug mode

Pipeline Building Blocks:
  build             Build the artifacts
  test              Run tests against your built application images
  deploy            Deploy pre-built artifacts
  delete            Delete any resources deployed by Skaffold
  render            Generate rendered Kubernetes manifests
  apply             Apply hydrated manifests to a cluster
  verify            Run verification tests against skaffold deployments

Getting Started With a New Project:
  init              Generate configuration for deploying an application
```

# 流水线

| **Pipeline stages**    | **Description**                                              |
| ---------------------- | ------------------------------------------------------------ |
| Init                   | generate a starting point for Skaffold configuration         |
| Build                  | build images with different builders                         |
| Render                 | render manifests with different renderers                    |
| Tag                    | tag images based on different policies                       |
| Test                   | run tests with testers                                       |
| Deploy                 | deploy with kubectl, kustomize or helm                       |
| Verify                 | verify deployments with specified test containers            |
| File Sync              | sync changed files directly to containers                    |
| Log Tailing            | tail logs from workloads                                     |
| Port Forwarding        | forward ports from services and arbitrary resources to localhost |
| Deploy Status Checking | wait for deployed resources to stabilize                     |
| Lifecycle Hooks        | run code triggered by different events during the skaffold process lifecycle |
| Cleanup                | cleanup manifests and images                                 |

阶段详细介绍参考官方文档 [1]。

# 案例

```yaml

```



# 参考链接

[1]: https://skaffold.dev/docs	"skaffold 官方文档"

