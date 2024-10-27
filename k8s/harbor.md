# 简介

<img src="../assets/harbor_1.png" alt="img" style="zoom: 25%;" />

- Nginx(Proxy)：用于代理 Harbor 的 registry、UI、token 等服务。
- db：负责储存用户权限、审计日志、Docker image 分组信息等数据。
- UI：提供图形化界面，帮助用户管理 registry 上的镜像, 并对用户进行授权。
- jobsevice：负责镜像复制工作的，他和 registry 通信，从一个 registry pull 镜像，然后 push 到另一个 registry ，并记录 job_log。
- adminserver：是系统的配置管理中心附带检查存储用量，ui 和 jobsevice 启动时候需要加载 adminserver 的配置。
- Registry：原生的 docker 镜像仓库，负责存储镜像文件。
- Log：为了帮助监控 Harbor 运行，负责收集其他组件的 log，记录到 syslog 中。

# 安装

## 前期准备

- Kubernetes 集群安装

- Helm 安装

- 动态供应，如：NFS、CEPH 等

- Ingress  安装
- TLS 证书

## 下载 Harbor

```shell
# 添加 chart 仓库
helm repo add harbor https://helm.goharbor.io

# 更新 chart 信息
helm repo update

# 下载 Harbor
helm pull harbor/harbor --version 1.7.1 --untar
```

## 修改配置

```yaml
expose:
  type: ingress
  tls:
    enabled: true
  ingress:
    hosts:
      core: harbor.zznode.com
      notary: notary.zznode.com
    annotations:
      kubernetes.io/ingress.class: "nginx"
      ingress.kubernetes.io/ssl-redirect: "true"
      ingress.kubernetes.io/proxy_timeout: "300s"
      ingress.kubernetes.io/proxy_connect_timeout: "5s"
      ingress.kubernetes.io/proxy-body-size: "0"
      ingress.kubernetes.io/client-max-body-size : "0"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
      nginx.ingress.kubernetes.io/proxy-body-size: "0"
      nginx.ingress.kubernetes.io/client-max-body-size: "0"
      nginx.ingress.kubernetes.io/proxy_timeout: "300s"
      nginx.ingress.kubernetes.io/proxy_connect_timeout: "5s"

externalURL: https://harbor.zznode.com

persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      storageClass: "csi-rbd-sc"
      size: 10Gi

    chartmuseum:
      storageClass: "csi-rbd-sc"
      size: 10Gi

    jobservice:
      storageClass: "csi-rbd-sc"
      size: 2Gi

    # If external database is used, the following settings for database will
    # be ignored
    database:
      storageClass: "csi-rbd-sc"
      size: 2Gi

    # If external Redis is used, the following settings for Redis will
    # be ignored
    redis:
      storageClass: "csi-rbd-sc"
      size: 3Gi

    trivy:
      storageClass: "csi-rbd-sc"
      size: 5Gi

  # 定义使用什么存储后端来存储镜像和 charts 包
  # 下面使用 s3 存储
  imageChartStorage:
    # 正对镜像和chart存储是否禁用跳转，对于一些不支持的后端(例如对于使用minio的`s3`存储)，需要禁用它。为了禁止跳转，只需要设置`disableredirect=true`即可
    disableredirect: true

    # 指定存储类型："filesystem", "azure", "gcs", "s3", "swift", "oss"，在相应的区域填上对应的信息。
    # 如果你想使用 pv 则必须设置成"filesystem"类型
    type: s3

    s3:
      region: Default Region
      bucket: harbor
      accesskey: 7X1X28GZNB9XGIFN2CLS
      secretkey: ywOkr9BYSHRwzGEgB5mCJpb8QpTn4w19QNseKSAj
      regionendpoint: http://10.1.19.60:7480
      secure: false
      #encrypt: false
      #keyid: mykeyid
      #v4auth: true
      #chunksize: "5242880"
      #rootdirectory: /s3/object/name/prefix
      #storageclass: STANDARD
```

## Helm 安装

```shell
# install
helm install harbor ./harbor  -f  values.yaml -n kube-ops
```

## 配置 Https

```shell
kubectl get secret -n kube-ops harbor-ingress -oyaml  > harbor-ingress.yaml

vim harbor-ingress.yaml
	# 修改其中的 tls.crt, tls.key
	# cat server.crt | base64 -w0    # 获取 tls.crt
	# cat server.key | base64 -w0    # 获取 tls.key 

# 使用修改的配置生效
kubectl apply -f harbor-ingress.yaml
```

## 验证

```shell
# 登录harbor
docker login harbor.zznode.com  

docker pull busybox:latest
docker tag busybox:latest harbor.zznode.com/library/busybox:v1
docker push harbor.zznode.com/library/busybox:v1

docker rmi -f harbor.zznode.com/library/busybox:v1
docker pull harbor.zznode.com/library/busybox:v1
```

> 如果是自制证书, 要在Docker Client 的启动文件里面加入 --insecure-registry 参数，强制信任这个镜像仓库。
> 将 harbor 仓库的地址添加到  `/etc/docker/daemon.json`   <<   "insecure-registries" : ["harbor.k8s.com"] 

# 使用

## 新建项目

Harbor 是以项目为单位的，所以需要新建项目。

## 推送镜像

由于 Harbor 使用的是 https 通信，所以需要让 Docker 信任这个 https （即 Docker 不信任自定义的域名（自签证书））。

```shell
# 复制 tls 证书
scp tls.crt [USERNAME]@[IP]:/etc/docker/certs.d/[DOMAIN_NAME]

# 重启 Docker
systemctl restart docker

# 登录 Docker
docker login -u [USERNAME] [REGISTRY]

# 推送镜像
docker tag [IMAGE]
docker push [IMAGE]
```

## 推送 Chart

```shell
# 创建 Chart
helm create [CHART]

# 打包 Chart
helm package [CHART_PATH] -d charts/ [--version 1.0.0]

# 登录 Harbor
helm registry login [--insecure host] -u xxx

# 推送
helm chart push [ref] [flags]
```

