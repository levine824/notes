# 基本概念

**PV 与 PVC**

- PVC：Pod 对存储的请求。PVC 会被 Pod 动态加载成为一个存储卷。

- PV：可以由运维手工分配，也可以使用 StorageClass 动态分配。PV 受 Kubernetes 管理，但并不与特定的 Pod 直接绑定。

- StorageClass：由管理员创建，可以用来动态的创建存储卷和 PV。

在 Kubernetes 中，PVC 是管理有状态应用的一个推荐方案。有了 PVC，Pod 可以申请并连接到存储卷，这些存储卷在 Pod 生命周期结束之后，还能独立存在。PVC 在存储方面让开发和运维的职责得以分离。运维人员负责供应存储，而开发人员则可以在不知后端细节的情况下，申请使用这些存储卷。

**Ceph**

存储可以分为：

- 物理存储：实际连接和加载的存储卷。

- 分布式存储：一个有效的解决有状态工作负载高可用问题的方案。

Ceph 就是一个分布式存储系统。

**Rook**

Rook 是一个编排器，能够支持包括 Ceph 在内的多种存储方案。Rook 简化了 Ceph 在 Kubernetes 集群中的部署过程。

# Ceph

## 功能 

- Ceph 对象存储：键值存储，其接口就是简单的 GET,PUT,DEL 等。

- Ceph 块设备：AWS 的 EBS ，Ceph 的 RBD (RBD 是 Ceph 面向块存储的接口)。

- Ceph 文件系统：它比块存储具有更丰富的接口，需要考虑目录、文件属性等支持，实现一个支持并行化的文件存储应该是最困难的。

## 架构

一个 Ceph 存储集群需要至少一个 Ceph 监视器、Ceph 管理器、Ceph OSD（对象存储守护程序）。如果需要运行 Ceph 文件系统客户端，则需要部署 Ceph Metadata Server。

### Monitors  

Ceph Monitor (ceph-mon) 监视器是维护集群状态信息 。维护集群状态的映射，包括监视器映射，管理器映射，OSD 映射，MDS 映射和 CRUSH 映射。这些映射是 Ceph 守护程序相互协调所必需的关键群集状态。监视器还负责管理守护程序和客户端之间的身份验证。通常至少需要三个监视器（这也是为什么需要三个 Node 节点的原因所在）才能实现冗余和高可用性。

###  Managers

Ceph Manager 守护进程(ceph-mgr) 是负责跟踪运行时指标和 Ceph 集群的当前状态。Ceph Manager 守护进程（ceph-mgr）负责跟踪运行时指标和 Ceph 集群的当前状态，包括存储利用率，当前性能指标和系统负载。Ceph Manager 守护程序还托管基于 python 的模块，以管理和公开 Ceph 集群信息，包括基于 Web 的 Ceph Dashboard 和 REST API 。通常，至少需要两个管理器才能实现高可用性。

### Ceph OSDs

Ceph OSD (对象存储守护进程, ceph-osd) 是存储数据的。通过检查其他 Ceph OSD 守护程序的心跳来存储数据，处理数据复制，恢复，重新平衡，并向 Ceph 监视器和管理器提供一些监视信息。 通常至少需要 3 个 Ceph OSD 才能实现冗余和高可用性。 

### MDSs

Ceph Metadata Server (MDS, ceph-mdsceph 元数据服务器) ，存储能代表 Ceph File System 的元数据(如：Ceph 块设备和 Ceph 对象存储不使用 MDS)。Ceph 元数据服务器允许 POSIX 文件系统用户执行基本命令（如 ls，find 等），而不会给 Ceph 存储集群带来巨大负担。

# Rook

Rook 是云原生平台的存储编排工具。

## 工作原理

![img](../assets/rook_1.png)

## 架构

![img](../assets/rook_2.png)

## Operator

k8s 中 Operator + CRD，可以快速帮我们部署一些有状态应用集群，如redis，mysql，Zookeeper 等。Rook 的 Operator 是 k8s 集群和存储集群之间进行交互的解析器。

## 安装

### 先决条件

- Kubernetes > v1.17

- Kubernetes 集群各节点主机内核版本不低于 4.17，各个节点主机需要安装 lvm2 。 

- Kubernetes 集群至少有 3 个工作节点，为了配置 Ceph 存储集群，至少需要以下本地存储选项之一： 
  - 原始设备（无分区或格式化文件系统）。
  - 原始分区（无格式化文件系统）。
  - block模式下存储类中可用的 PV 。

### 部署 Operator

```shell
# 下载
wget https://github.com/rook/rook/archive/refs/tags/v1.9.2.zip

# 解压
unzip v1.9.2.zip
```

修改 `operator.yaml` 文件：

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: rook-ceph-operator-config
  namespace: rook-ceph 
data:
  ROOK_LOG_LEVEL: "INFO"
  ROOK_CSI_ENABLE_CEPHFS: "true"
  ROOK_CSI_ENABLE_RBD: "true"
  ROOK_CSI_ENABLE_NFS: "false"
  ROOK_CSI_ENABLE_GRPC_METRICS: "false"
  CSI_ENABLE_ENCRYPTION: "false"
  CSI_PROVISIONER_REPLICAS: "2"
  CSI_ENABLE_CEPHFS_SNAPSHOTTER: "true"
  CSI_ENABLE_RBD_SNAPSHOTTER: "true"
  CSI_FORCE_CEPHFS_KERNEL_CLIENT: "true"
  CSI_RBD_FSGROUPPOLICY: "ReadWriteOnceWithFSType"
  CSI_CEPHFS_FSGROUPPOLICY: "ReadWriteOnceWithFSType"
  CSI_NFS_FSGROUPPOLICY: "ReadWriteOnceWithFSType"
  ROOK_CSI_ALLOW_UNSUPPORTED_VERSION: "false"
  CSI_PLUGIN_ENABLE_SELINUX_HOST_MOUNT: "false"
  # 原来
  # these images to the desired release of the CSI driver.
  # ROOK_CSI_CEPH_IMAGE: "quay.io/cephcsi/cephcsi:v3.6.1"
  # ROOK_CSI_REGISTRAR_IMAGE: "k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.5.0"
  # ROOK_CSI_RESIZER_IMAGE: "k8s.gcr.io/sig-storage/csi-resizer:v1.4.0"
  # ROOK_CSI_PROVISIONER_IMAGE: "k8s.gcr.io/sig-storage/csi-provisioner:v3.1.0"
  # ROOK_CSI_SNAPSHOTTER_IMAGE: "k8s.gcr.io/sig-storage/csi-snapshotter:v5.0.1"
  # ROOK_CSI_ATTACHER_IMAGE: "k8s.gcr.io/sig-storage/csi-attacher:v3.4.0"
  # ROOK_CSI_NFS_IMAGE: "k8s.gcr.io/sig-storage/nfsplugin:v3.1.0"
  # 打开注释，进行修改
  # ------------ 修改 -------------
  ROOK_CSI_CEPH_IMAGE: "ccr.ccs.tencentyun.com/cephcsi/cephcsi:v3.6.1"
  ROOK_CSI_REGISTRAR_IMAGE: "ccr.ccs.tencentyun.com/sig-storage/csi-node-driver-registrar:v2.5.0"
  ROOK_CSI_RESIZER_IMAGE: "ccr.ccs.tencentyun.com/sig-storage/csi-resizer:v1.4.0"
  ROOK_CSI_PROVISIONER_IMAGE: "ccr.ccs.tencentyun.com/sig-storage/csi-provisioner:v3.1.0"
  ROOK_CSI_SNAPSHOTTER_IMAGE: "ccr.ccs.tencentyun.com/sig-storage/csi-snapshotter:v5.0.1"
  ROOK_CSI_ATTACHER_IMAGE: "ccr.ccs.tencentyun.com/sig-storage/csi-attacher:v3.4.0"
  ROOK_CSI_NFS_IMAGE: "ccr.ccs.tencentyun.com/sig-storage/nfsplugin:v3.1.0"
  # ------------ 修改 -------------

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rook-ceph-operator
  namespace: rook-ceph 
  labels:
    operator: rook
    storage-backend: ceph
    app.kubernetes.io/name: rook-ceph
    app.kubernetes.io/instance: rook-ceph
    app.kubernetes.io/component: rook-ceph-operator
    app.kubernetes.io/part-of: rook-ceph-operator
spec:
  selector:
    matchLabels:
      app: rook-ceph-operator
  replicas: 1
  template:
    metadata:
      labels:
        app: rook-ceph-operator
    spec:
      serviceAccountName: rook-ceph-system
      containers:
        - name: rook-ceph-operator
          # ------------ 修改 -------------
          image: ccr.ccs.tencentyun.com/k8s-rook/ceph:v1.9.2 # 原来 rook/ceph:v1.9.2
          # ------------ 修改 -------------
          args: ["ceph", "operator"]
          securityContext:
            runAsNonRoot: true
            runAsUser: 2016
            runAsGroup: 2016
```

安装：

```shell
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
```

### 部署集群

修改 `cluster.yaml`：

```yaml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph 
spec:
  cephVersion:
    # ------------ 修改 -------------
    image: ccr.ccs.tencentyun.com/k8s-ceph/ceph:v16.2.7 # 原来是 quay.io/ceph/ceph:v16.2.7
    # ------------ 修改 -------------
    allowUnsupported: false
  dataDirHostPath: /var/lib/rook
  skipUpgradeChecks: false
  continueUpgradeAfterChecksEvenIfNotHealthy: false
  waitTimeoutForHealthyOSDInMinutes: 10
  mon:
    count: 3
    allowMultiplePerNode: false
  mgr:
    count: 2 
    allowMultiplePerNode: false
    modules:
      - name: pg_autoscaler
        enabled: true
  dashboard:
    enabled: true
    ssl: true
  monitoring:
    enabled: false
  network:
    connections:
      encryption:
        enabled: false
      compression:
        enabled: false
  crashCollector:
    disable: false
  cleanupPolicy:
    confirmation: ""
    sanitizeDisks:
      method: quick
      dataSource: zero
      iteration: 1
    allowUninstallWithVolumes: false
  annotations:
  labels:
  resources:
  removeOSDsIfOutAndSafeToRemove: false
  priorityClassNames:
    mon: system-node-critical
    osd: system-node-critical
    mgr: system-cluster-critical
  storage: # cluster level storage configuration and selection
    # ------------ 修改 -------------
    useAllNodes: false
    useAllDevices: false
    config:
      osdsPerDevice: '3' # 每个设备的 osd 数量
    nodes:
      - name: "k8s-node1" # 必须符合 kubernetes.io/hostname ，通过 kubectl get nodes --show-labels 查看
        devices: 
          - name: "sdb"
      - name: "k8s-node2"
        devices: 
         - name: "sdb"
      - name: "k8s-node3"
        devices: 
         - name: "sdb" 
    # ------------ 修改 -------------     
    onlyApplyOSDPlacement: false
```

安装

```shell
kubectl create -f cluster.yaml
```

### 部署 dashboard

```shell
# 安装 NodePort 形式
kubectl apply -f dashboard-external-https.yaml

# 获取的 dashboard 的密码
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 -d
```

注：Helm 安装此处省略。

## 使用

Ceph 分为块存储（RBD）和共享存储（CephFS）两类。 

- 块存储（RBD）：一般用于单节点读写（RWO，ReadWriteOnce），适用于有状态应用。

- 共享存储（CephFs）：一般用于多节点读写（RWX，ReadWriteMany），适用于无状态应用。

Rook 可以帮准我们创建好 StorageClass，我们只需要在 PVC 中指定存储类，Rook 就会调用 StorageClass 里面的 Provisioner供应商，接下来对 Ceph 集群操作。

### 配置 RBD

修改`rook-ceph-block.yaml`： 

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph # namespace:cluster
spec:
  failureDomain: host
  replicated:
    size: 3
    # Disallow setting pool with replica 1, this could lead to data loss without recovery.
    # Make sure you're *ABSOLUTELY CERTAIN* that is what you want
    requireSafeReplicaSize: true
    # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
    # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
    #targetSizeRatio: .5
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  # clusterID is the namespace where the rook cluster is running
  # If you change this namespace, also change the namespace below where the secret namespaces are defined
  clusterID: rook-ceph # namespace:cluster

  # If you want to use erasure coded pool with RBD, you need to create
  # two pools. one erasure coded and one replicated.
  # You need to specify the replicated pool here in the `pool` parameter, it is
  # used for the metadata of the images.
  # The erasure coded pool must be set as the `dataPool` parameter below.
  #dataPool: ec-data-pool
  pool: replicapool

  # (optional) mapOptions is a comma-separated list of map options.
  # For krbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
  # For nbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
  # mapOptions: lock_on_read,queue_depth=1024

  # (optional) unmapOptions is a comma-separated list of unmap options.
  # For krbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
  # For nbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
  # unmapOptions: force

   # (optional) Set it to true to encrypt each volume with encryption keys
   # from a key management system (KMS)
   # encrypted: "true"

   # (optional) Use external key management system (KMS) for encryption key by
   # specifying a unique ID matching a KMS ConfigMap. The ID is only used for
   # correlation to configmap entry.
   # encryptionKMSID: <kms-config-id>

  # RBD image format. Defaults to "2".
  imageFormat: "2"

  # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
  imageFeatures: layering

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster
  # Specify the filesystem type of the volume. If not specified, csi-provisioner
  # will set default as `ext4`. Note that `xfs` is not recommended due to potential deadlock
  # in hyperconverged settings where the volume is mounted on the same node as the osds.
  csi.storage.k8s.io/fstype: ext4
# uncomment the following to use rbd-nbd as mounter on supported nodes
# **IMPORTANT**: CephCSI v3.4.0 onwards a volume healer functionality is added to reattach
# the PVC to application pod if nodeplugin pod restart.
# Its still in Alpha support. Therefore, this option is not recommended for production use.
#mounter: rbd-nbd
allowVolumeExpansion: true
reclaimPolicy: Delete
```

```shell
kubectl apply -f rook-ceph-block.yaml
```

### 配置 CephFS

修改 `rook-cephfs.yaml`:

```yaml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph # namespace:cluster
spec:
  # The metadata pool spec. Must use replication.
  metadataPool:
    replicated:
      size: 3
      requireSafeReplicaSize: true
    parameters:
      # Inline compression mode for the data pool
      # Further reference: https://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression
      compression_mode:
        none
        # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
      # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
      #target_size_ratio: ".5"
  # The list of data pool specs. Can use replication or erasure coding.
  dataPools:
    - name: replicated
      failureDomain: host
      replicated:
        size: 3
        # Disallow setting pool with replica 1, this could lead to data loss without recovery.
        # Make sure you're *ABSOLUTELY CERTAIN* that is what you want
        requireSafeReplicaSize: true
      parameters:
        # Inline compression mode for the data pool
        # Further reference: https://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression
        compression_mode:
          none
          # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
        # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
        #target_size_ratio: ".5"
  # Whether to preserve filesystem after CephFilesystem CRD deletion
  preserveFilesystemOnDelete: true
  # The metadata service (mds) configuration
  metadataServer:
    # The number of active MDS instances
    activeCount: 1
    # Whether each active MDS instance will have an active standby with a warm metadata cache for faster failover.
    # If false, standbys will be available, but will not have a warm cache.
    activeStandby: true
    # The affinity rules to apply to the mds deployment
    placement:
      #  nodeAffinity:
      #    requiredDuringSchedulingIgnoredDuringExecution:
      #      nodeSelectorTerms:
      #      - matchExpressions:
      #        - key: role
      #          operator: In
      #          values:
      #          - mds-node
      #  topologySpreadConstraints:
      #  tolerations:
      #  - key: mds-node
      #    operator: Exists
      #  podAffinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
                - key: app
                  operator: In
                  values:
                    - rook-ceph-mds
            # topologyKey: kubernetes.io/hostname will place MDS across different hosts
            topologyKey: kubernetes.io/hostname
        preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - rook-ceph-mds
              # topologyKey: */zone can be used to spread MDS across different AZ
              # Use <topologyKey: failure-domain.beta.kubernetes.io/zone> in k8s cluster if your cluster is v1.16 or lower
              # Use <topologyKey: topology.kubernetes.io/zone>  in k8s cluster is v1.17 or upper
              topologyKey: topology.kubernetes.io/zone
    # A key/value list of annotations
    # annotations:
    #  key: value
    # A key/value list of labels
    # labels:
    #  key: value
    # resources:
    # The requests and limits set here, allow the filesystem MDS Pod(s) to use half of one CPU core and 1 gigabyte of memory
    #  limits:
    #    cpu: "500m"
    #    memory: "1024Mi"
    #  requests:
    #    cpu: "500m"
    #    memory: "1024Mi"
    priorityClassName: system-cluster-critical
    livenessProbe:
      disabled: false
    startupProbe:
      disabled: false
  # Filesystem mirroring settings
  # mirroring:
    # enabled: true
    # list of Kubernetes Secrets containing the peer token
    # for more details see: https://docs.ceph.com/en/latest/dev/cephfs-mirroring/#bootstrap-peers
    # peers:
      #secretNames:
        #- secondary-cluster-peer
    # specify the schedule(s) on which snapshots should be taken
    # see the official syntax here https://docs.ceph.com/en/latest/cephfs/snap-schedule/#add-and-remove-schedules
    # snapshotSchedules:
    #   - path: /
    #     interval: 24h # daily snapshots
    #     startTime: 11:55
    # manage retention policies
    # see syntax duration here https://docs.ceph.com/en/latest/cephfs/snap-schedule/#add-and-remove-retention-policies
    # snapshotRetention:
    #   - path: /
    #     duration: "h 24"
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
  # clusterID is the namespace where the rook cluster is running
  # If you change this namespace, also change the namespace below where the secret namespaces are defined
  clusterID: rook-ceph # namespace:cluster

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-replicated

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster

  # (optional) The driver can use either ceph-fuse (fuse) or ceph kernel client (kernel)
  # If omitted, default volume mounter will be used - this is determined by probing for ceph-fuse
  # or by setting the default mounter explicitly via --volumemounter command-line argument.
  # mounter: kernel
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  # uncomment the following line for debugging
  #- debug
```

```shell
kubectl apply -f rook-cephfs.yaml
```

## 卸载

```shell
# 卸载之前的 Cephcluster CRD
kubectl -n rook-ceph delete cephcluster rook-ceph

# 删除 Operator 和相关的资源
kubectl delete -f rook-cephfs.yaml 
kubectl delete -f rook-ceph-block.yaml
kubectl delete -f cluster.yaml
kubectl delete -f operator.yaml
kubectl delete -f common.yaml
kubectl delete -f crds.yaml

# 删除各节点数据
rm -rf /var/lib/rook

# 擦除硬盘数据
###
#!/usr/bin/env bash
DISK="/dev/sdb"

sgdisk --zap-all $DISK

dd if=/dev/zero of="$DISK" bs=1M count=100 oflag=direct,dsync

blkdiscard $DISK

ls /dev/mapper/ceph-* | xargs -I% -- dmsetup remove %

rm -rf /dev/ceph-*
rm -rf /dev/mapper/ceph--*
###
```

# 参考链接

[1]: https://docs.ceph.com/en/latest/	"Ceph 官网"
[2]: https://rook.github.io/docs/rook/latest-release/Getting-Started/intro/	"Rook 官网"

