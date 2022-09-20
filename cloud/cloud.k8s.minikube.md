
![](https://s3.leryn.top/website/image/minikube.png#crop=0&crop=0&crop=1&crop=1&height=243&id=vyoa5&originHeight=1552&originWidth=1600&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=250)
<a name="THAKk"></a>
# Minikube - Kubernetes 集群部署
Minikube 是一个单机 Kubernetes 的解决方案, 以下称为`minikube`.

参考文档:

- [Minikube - 官方入门文档](https://minikube.sigs.k8s.io/docs/start/)
<a name="zGwrq"></a>
## 二进制安装
<a name="h9h1t"></a>
### 安装 kubectl

`kubectl` 这个命令在`minikube`中是可选的, 这里默认选择安装. 如果不安装的话, 后续可以使用复杂的命令代替. 这里顺便把三个客户端工具`kubectl`, `kubelet`, `kubeadm`都安装上.<br />参考[K8S - 官方文档](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)

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

如果`apt-get update`报错, 无法信任阿里云和腾讯云的 apt-key<br />只需要将第一句命令的最后的密钥换成报错信息中提示缺少的密钥即可, 然后重新 update

```bash
sudo apt-get update
sudo apt install -y kubectl kubelet kubeadm
```

<a name="ocw6G"></a>
### 前置准备

- Linux 环境
- 2 核 CPU
- 2GB 内存
- 20GB 硬盘空间
- 网络连接
- 虚拟化容器/虚拟机: Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMWare

<a name="B01tV"></a>
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
