# 简介

Skaffold 是由 Google 发布的命令行工具，专注于促进 K8S 应用的持续 deployment。自动化 building 和 deploying 到 k8s 集群的任务，可以让开发者专注于编写代码。

2019年11月份，Skaffold 普遍可用的版本发布了，承诺自动化开发工作流程，以此来节省开发者的时间。那么，Skaffold 提供哪些功能呢？

- 当你开发的时候，检测代码的变动；
- 基于你的 Dockerfile 或者 Jib 自动化 build 和创建你的 artifacts（也就是 Docker image）；
- 给 artifacts 打 tag；
- 把 artifacts 发布/部署到你的 kubernetes 集群。

**先决条件**

Skaffold 需要先安装 minikube/kubernetes 和 kubectl。

# 安装

```shell
# 安装 minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --force --driver=docker
alias kubectl="minikube kubectl --"
# 安装 skaffold
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64
```

