---
id: k8s.kind
tags:
- k3s
- k8s
- kubernetes
title: Kind - Kubernetes in Docker

---
# Kind - Kubernetes in Docker
参考文档：

+ [kind – Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)

Kind（Kubernetes IN Docker）是一个用于快速创建测试的 Kubernetes，仅仅需要安装 docker 或者 podman 即可。非常适合在本地搭建集群进行 e2e 测试。

## 安装步骤
前置条件：

+ 需要额外安装 docker 或者 podman

只需要手动安装二进制即可：

```bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-arm64

chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

如下命令，即可创建一个默认集群：

```bash
kind create cluster
```

加载本地 docker 镜像：

```bash
kind load docker-image debian:latest
```

## 创建集群
如果需要创建一个定制化的集群，那么可以用过编写配置文件，例如：

+ 需要 1 个 master 和 1 个 worker 节点
+ 需要 1 个节点标记成 CPU 计算节点，另一个节点标记成 GPU 节点
+ 通过 hostPort 访问集群内的 ingress 的 30080/30443 端口

你还可以禁用 kind 默认的 CNI 插件等等，这里不举例了。

```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
name: kind
nodes:
- role: control-plane
  labels:
    node-type: cpu
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 30080
    hostPort: 30080
    protocol: TCP
  - containerPort: 30443
    hostPort: 30443
    protocol: TCP
- role: worker
  labels:
    node-type: gpu
```

```yaml
kind create cluster --config cluster-config.yaml
```

## 原理
最后介绍一下 Kind 的核心原理：

Kind 用 container 来模拟节点，在节点里跑 systemd 用其托管 containerd 和 kubelet，再由 kubelet 调起 api-server，etcd，cni 等等。

镜像分为 node 镜像和 base 镜像。

### node 镜像
node 镜像的构建比较复杂，需要通过运行 base 镜像，并在 base 镜像内执行操作，再保存此容器内容为镜像的方式来完成构建。它包含的操作有：

+ 构建 Kubernetes 相关资源（比如二进制文件和镜像）
+ 运行一个用于构建的容器
+ 把构建的 Kubernetes 相关资源复制到容器里
+ 调整部分组件配置参数，以支持在容器内运行
+ 预先拉去运行环境需要的镜像
+ 通过 docker commit 方式保存当前的构建容器为 node 镜像

### base 镜像
base 镜像目前使用了 ubuntu 作为基础镜像，做了以下调整

+ 安装 systemd 相关的包，并调整一些配置以适应在容器内运行
+ 安装 Kubernetes 运行时的依赖包，比如 conntrack、socat、CNI
+ 安装容器
+ 运行环境，比如 Containerd、crictl
+ 配置自己的 ENTRYPOINT 脚本，以适应和调整容器内运行的问题

### 创建集群
Kind 创建集群的基本过程为

+ 根据传入的参数创建 container，分为 control node 和 worker node 两种（如果是 HA master，还有一个loadbalancer node）
+ 如果需要，配置 loadbalancer 的配置，主要是 Nginx 配置文件
+ 生成 kubeadm 配置 
+ 对于第一个控制节点，使用 kubeadm init 初始化单节点集群
+ 配置安装 CNI 插件
+ 配置存储（实际是安装了一个使用 hostpath 的 storageclass）
+ 其他的控制节点，通过 kubeadm join --experimental-control-plane 的方式扩容控制节点
+ 通过 kubeadm join 扩容其他的工作节点
+ 等待集群创建完成
+ 生成访问配置，打印使用帮助具体的创建流程

  


