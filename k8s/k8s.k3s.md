---
id: k8s.k3s
tags: []
title: K3S & Cilium

---
# K3S & Cilium 部署
参考文档：

+ [K3S 安装文档 - 官方](https://docs.rancher.cn/docs/k3s/installation/install-options/_index)
+ [https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/)

K3S 是 Rancher 官方提出的轻量级 Kubernetes，阉割或降级了很多 Kubernetes 的配置，例如用 SQLite 来替换 Etcd，但它仍然满足 Kubernetes 的所有 API。性能上，资源占用比 Minikube 更小，是 Kubernetes 开发机最合适的产品实现。

CNI 改用 cilium 而不是 flannel 等。

## 安装步骤
运行以下命令即可一键安装，在这之前解释一下参数的意义：

+ 参数`INSTALL_K3S_EXEC="--no-deploy traefik "`表示禁用安装默认的 traefik 网关。之后我们会用 ingress-nginx-controller 来代替 traefik。如果你更熟悉 traefik，可以继续使用，但本人更加熟悉 nginx。**<font style="color:#DF2A3F;">新版本</font>**中已经被替换为参数 `INSTALL_K3S_EXEC="--disable traefik --disable servicelb"`了
+ 默认使用 containerd 作为容器运行时，如果需要使用 docker 作为容器运行时，在 `sh -s - ` 后加上 `--docker` 参数
+ 参数`INSTALL_K3S_EXEC="--flannel-backend=none --disable-network-policy "`表示禁用 flannel 作为集群的 CNI 插件，如果需要安装其他 CNI 时则需要禁用这两项<font style="background-color:#FBDE28;"></font>
+ `--service-node-port-range=1-65535` 表示开放默认 Service NodePort 端口范围到任意端口

```bash
# 一键安装k3s
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--no-deploy traefik" sh -s - --docker

# 国内加速使用
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | \
  INSTALL_K3S_MIRROR=cn \
  INSTALL_K3S_EXEC="--disable traefik --disable servicelb --flannel-backend=none --disable-network-policy " \
  sh -s - --service-node-port-range=1-65535
```

```bash
# 查看版本
sudo k3s kubectl version

# 查看node
sudo k3s kubectl get nodes

# 查看pods
sudo k3s kubectl get pods
```

其实安装之后 `/usr/local/bin` 下有了这几个命令：

```bash
/usr/local/bin/k3s
/usr/local/bin/k3s-killall.sh
/usr/local/bin/k3s-uninstall.sh
/usr/local/bin/kubectl -> k3s
```

## 安装 CNI
安装 Cilium 作为 CNI。先安装 `cilium-cli` 命令行可执行。

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/master/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

直接安装即可，等待安装完成后，即可：

```bash
cilium install
cilium status --wait

# (可选) 开启 Web UI
cilium ui enable
```

## 配置文件
一些常用的文件路径：

```bash
# Kubeconfig
/etc/rancher/k3s/k3s.yaml

# Containerd 配置文件
/var/lib/rancher/k3s/agent/etc/containerd/config.toml

# crictl 配置文件
/var/lib/rancher/k3s/agent/etc/crictl.yaml

# SQLite 持久化文件, 及时备份
/var/lib/rancher/k3s/server/db/state.db
```

## 卸载 K3S
如果您使用安装脚本安装了k3s，那么在安装过程中会生成一个卸载 k3s 的脚本。卸载 k3s 会删除集群数据和所有脚本。要使用不同的安装选项重新启动集群，请使用不同的标志重新运行安装脚本。

要从 server 节点卸载 k3s，请运行：

```bash
/usr/local/bin/k3s-uninstall.sh
```

要从 agent 节点卸载 k3s，请运行：

```bash
/usr/local/bin/k3s-agent-uninstall.sh
```

## Helm支持
正常安装 K3S 无法直接使用 Helm：

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```



# Minikube
Minikube 是一个单机 Kubernetes 的解决方案，以下称为`minikube`。



参考文档：

+ [Minikube - 官方入门文档](https://minikube.sigs.k8s.io/docs/start/)

## 二进制安装
### 安装 kubectl


`kubectl` 这个命令在`minikube`中是可选的，这里默认选择安装。如果不安装的话，后续可以使用复杂的命令代替。这里顺便把三个客户端工具`kubectl`，`kubelet`，`kubeadm`都安装上。

参考[K8S - 官方文档](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)



```bash
# 添加apt key
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6A030B21BA07F4FB
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys FEEA9169307EA071
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8B57C5C2836F4BEB
```



```bash
# 配置镜像, k8s官方镜像可能由于网络原因无法访问请跳过
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
#deb https://apt.kubernetes.io/ kubernetes-xenial main
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
deb https://mirrors.cloud.tencent.com/kubernetes/apt kubernetes-xenial main
EOF
```



如果`apt-get update`报错, 无法信任阿里云和腾讯云的 apt-key

只需要将第一句命令的最后的密钥换成报错信息中提示缺少的密钥即可, 然后重新 update



```bash
sudo apt-get update
sudo apt install -y kubectl kubelet kubeadm
```



### 前置准备
+ Linux 环境
+ 2 核 CPU
+ 2GB 内存
+ 20GB 硬盘空间
+ 网络连接
+ 虚拟化容器/虚拟机: Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMWare



### 安装步骤


```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```



启动集群, 需要 root 权限但不能是 root. 这个安装速度真的极其慢!!



```bash
minikube start \
  --image-mirror-country=cn \
  --registry-mirror=https://registry.docker-cn.com \
  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```



使用`kubectl`查看 k8s 集群. 可能会需要用到`kubectl`命令.



```bash
# 需要安装kubectl
kubectl get po -A

# 如果不想安装的话
minikube kubectl -- get po -A
```



需要需要仪表盘来查看集群状态的话, 可以输入如下命令:



```bash
# 这一步其实不用做, 后续我们会用Rancher来管理k8s集群
minikube dashboard
```

