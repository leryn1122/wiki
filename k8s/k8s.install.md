---
id: k8s.install
tags:
- k8s
- kubernetes
title: "Kubernetes \u5B89\u88C5\u624B\u518C"

---
# Kubernetes 安装手册


### 前置准备
前置准备：

+ 安装前置依赖
+ 安装容器运行时，已经不推荐使用 Docker，请使用 containerd 或其他运行时

安装前置依赖：

```bash
sudo apt update && sudo apt install -y \
  apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

```plain
buildkit-v0.16.0.linux-amd64.tar.gz
cilium-linux-amd64.tar.gz
cni-plugins-linux-amd64-v1.6.0.tgz
containerd-1.7.23-linux-amd64.tar.gz
kubeadm
kubectl
kubelet
k9s_Linux_amd64.tar.gz
nerdctl-1.7.7-linux-amd64.tar.gz
runc.amd64
```

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

```bash
modprobe br_netfilter
lsmod | grep br_netfilter
```

关闭 swap 分区

如果开启 swap 分区，当节点 OOM 时内存被交换到磁盘上时，这个节点可能会 hang up，而且没有任何报错信息提示，甚至无法使用物理终端访问，最后只能硬重启整个节点。所以 kubelet 启动时会检测 swap 是否关闭，如果没有关闭则会启动失败。

```bash
swapoff -a

vim /etc/fstab
# 注释掉 swap 分区
```

### 二进制安装
```bash
vim /etc/containerd/config.toml
```

```bash
vim /usr/local/lib/systemd/system/containerd.service
```

```toml
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

nerdctl 的配置文件

```bash
vim /etc/nerdctl/nerdctl.toml
```

```toml
# This is an example of /etc/nerdctl/nerdctl.toml .
# Unrelated to the daemon's /etc/containerd/config.toml .

debug          = false
debug_full     = false
address        = "unix:///run/containerd/containerd.sock"
namespace      = "k8s.io"
# snapshotter    = "stargz"
# cgroup_manager = "cgroupfs"
# hosts_dir      = ["/etc/containerd/certs.d", "/etc/docker/certs.d"]
experimental   = true
```

设置 buildkit

```bash
vim /usr/local/lib/systemd/system/buildkit.socket
vim /usr/local/lib/systemd/system/buildkit.service
```

```toml
[Unit]
Description=BuildKit
Documention=https://github.com/moby/buildkit
 
[Socket]
ListenStream=%t/buildkit/buildkitd.sock
 
[Install]
WantedBy=sockets.target
```

```toml
[Unit]
Description=BuildKit
Documention=https://github.com/moby/buildkit
 
[Socket]
ListenStream=%t/buildkit/buildkitd.sock
 
[Install]
WantedBy=sockets.target
root@faraday-deb12-notebook-pc-bms:/data/installation# cat /usr/local/lib/systemd/system/buildkit.service 
[Unit]
Description=BuildKit
Require=buildkit.socket
After=buildkit.socket
Documention=https://github.com/moby/buildkit
 
[Service]
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true
 
[Install]
WantedBy=multi-user.target
```

### 高可用集群安装
如果一个 Kubernetes 要满足高可用，那么建议保证集群有：

+ 至少 3 个 master 节点
+ 至少 3 个 worker 节点
+ 外部的 SLB：例如 HAProxy + KeepAlived：
    - 至少 2 个 HAProxy 和至少 2 个 KeepAlived，建议和节点分离部署
    - 一个额外的空闲 IP

参考文档：

+ [HAProxy & KeepAlived](https://www.yuque.com/leryn/wiki/lbs.haproxy)

#### HAProxy 安装
```bash
sudo apt update && sudo apt install -y \
  haproxy
```

修改配置文件：

```bash
vim /etc/haproxy/haproxy.cfg
```

在默认配置后面追加 `master:6443`，`worker:80/443` 的高可用：

```plain
  ...

frontend my-k8s-master
  bind 0.0.0.0:6443
  mode tcp
  option tcplog
  timeout client 30000
  default_backend my-k8s-master

backend my-k8s-master
  mode tcp
  option tcplog
  option tcp-check
  timeout server 30000
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server apiserver1 k8s-master-IP地址:6443 check
    server apiserver2 k8s-master-IP地址:6443 check
    server apiserver3 k8s-master-IP地址:6443 check

frontend my-k8s-worker-http
  bind 0.0.0.0:80
  mode tcp
  option tcplog
  timeout client 30000
  default_backend my-k8s-worker-http

backend my-k8s-worker-http
  mode tcp
  option tcplog
  option tcp-check
    timeout server 30000
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server worker1 k8s-worker-IP地址:80 check
    server worker2 k8s-worker-IP地址:80 check
    server worker3 k8s-worker-IP地址:80 check

frontend my-k8s-worker-https
  bind 0.0.0.0:443
  mode tcp
  option tcplog
  timeout client 30000
  default_backend my-k8s-worker-https

backend my-k8s-worker-https
  mode tcp
  option tcplog
  option tcp-check
    timeout server 30000
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server worker1 k8s-worker-IP地址:443 check
    server worker2 k8s-worker-IP地址:443 check
    server worker3 k8s-worker-IP地址:443 check

```

启动 HAProxy：

```bash
# 检查配置文件正确性
haproxy -c -f /etc/haproxy/haproxy.cfg

# 启动 HAProxy
systemctl enable haproxy
systemctl restart haproxy
systemctl status haproxy
```

#### KeepAlived 安装
KeepAlived 用于占用 VIP（虚拟IP），不需要实际的服务器，但要占用一个空闲 IP。KeepAlived 会轮流占用这个 VIP，如果当前 KeepAlived 宕机，那么 VIP 会漂移到另一个 KeepAlived 之上，实现高可用。

```bash
sudo apt update && sudo apt install -y \
  keepalived
```

更改配置文件：

```bash
vim /etc/keepalived/keepalived.conf
```

```plain
! Configuration File for keepalived

# 多网卡服务器请写多个 Instance, 并注意更换 ID, 网卡名, IP 地址
vrrp_instance VI_1 {
  state BACKUP             
  interface ens6
  virtual_router_id 61
  priority 90
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  virtual_ipaddress {
    漂移IP地址-前置网段/24
  }
}
```

启动 KeepAlived：

```bash
systemctl enable  keepalived
systemctl restart keepalived
systemctl status  keepalived
```

然后我们初始化集群之前要将 Controll Plane 域名解析指向这个 VIP 的地址。

### 初始化集群
初始化集群：如果执行成功，控制台会打印 `kubeadm join` 命令，依次在对应的节点上运行：

+ 第一条是加入 master 节点使用的
+ 第二条是加入 worker 节点使用的

```bash
kubeadm init \
  --control-plane-endpoint master-test-k8s-home-shanghai.leryn.io \
  --cri-socket=unix:///run/containerd/containerd.sock \
  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
  --service-cidr=20.96.0.0/12 \
  --pod-network-cidr=30.96.0.0/12 \
  --ignore-preflight-errors=all \
  --upload-certs \
  --v=5
```

如果报错了可以随时清除设置并重新初始化：

```bash
kubeadm reset
systemctl daemon-reload
systemctl restart kubelet
```

```plain
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join master-test-k8s-home-shanghai.leryn.io:6443 --token 4c5cux.99s7drifrftiw998 \
	--discovery-token-ca-cert-hash sha256:ccb32ca5ab95512fbe9f3629192a3d2ed4fc8d634cbbd4771acc89f415f1bed5 \
	--control-plane --certificate-key a27cfde0f2ebaf81376fac39545a0296a529d4397fe842f2bbc27bce1de59dbd

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join master-test-k8s-home-shanghai.leryn.io:6443 --token 4c5cux.99s7drifrftiw998 \
	--discovery-token-ca-cert-hash sha256:ccb32ca5ab95512fbe9f3629192a3d2ed4fc8d634cbbd4771acc89f415f1bed5
```

初始化成功后，根据提示生产配置文件：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

如果超过 24 小时，Token 会自动过期，用以下命令生成新的 Token：

```bash
kubeadm token create --print-join-command
```

初始化集群后，会产生静态 Pod。这些 Pod 的 Manifest 文件会存在于指定目录下，标准的 Kubernetes 是 `/etc/kubernetes/manifests`。Kubernetes 始终保证这些 Pod 会存在，如果它们被误删除，过一段时间后也将重新生成。Kubernetes 内部组件的配置在这里修改：

```bash
# APIServer 开启8080端口 所有master节点都修改
vim /etc/kubernetes/manifests/kube-apiserver.yaml
# 修改内容: --insecure-port=8080 

# kubectl get cs 状态是 unhealthy
vim /etc/kubernetes/manifests/kube-scheduler.yaml
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
# 删除下面文件的配置 port=0 这一行

#  集群开放 30000 以下的端口, 需要手动修改 kubelet 的配置文件
vim /etc/kubernetes/manifests/kube-apiserver.yaml 
# 添加内容: --service-node-port-range=1-65535
```

安装 CNI 网络插件：

+ 虽然不安装网络插件无法让 Kubernetes 集群通讯，但是 Kubernetes 官方认为 CNI 不是它的范围，也没有提供默认的网络设施。
+ CNI 网络插件实现非常多：weave，flannel，calico 等等。这里安装 Weave，因为我司用的 Weave，但它不是目前最好的实现。

```plain
cilium install \
  --set kubeProxyReplacement=strict
```

其他 CNI：

weave：[https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation)

flannel：[https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml](https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml)

calico：[https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)

```bash
# 这个地址似乎近期失效了
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# 也可以用以下地址下载对应版本
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

安装完成后，验证集群：

```bash
# 查看节点状态, 全是 ready 就对了
kubectl get nodes

# 查看集群健康状况
kubectl get cs
```

# Kubernetes 运维
## Kubernetes 运维常见场景
### 集群证书问题
目前 Kubernetes 是由 `kubeadm` 安装的 1.18 版本，默认证书有效期是1年

通过以下命令统一更新由 `kubeadm` 管理的所有证书：

```bash
kubeadm alpha certs renew all
```

注意此时 Kubenetes 组件 api-server，scheduler 以及 controller manager 镜像内部的证书未更新（可能是kubernetes 1.18版本bug）

我们需要重新启动镜像：

```bash
docker ps | grep -E 'k8s_kube-apiserver|k8s_kube-controller-manager|k8s_kube-scheduler' | awk -F ' ' '{print $1}' | xargs docker restart
```

如果未重启，此后证书过期，将导致 Pod 调度等问题。此时通过docker logs命令可观察到上述容器报告以下错误：

> Unable to authenticate the request due to an error: x509: certificate has expired or is not yet valid
>

再次检查证书：

```bash
kubeadm alpha certs check-expiration
```

### 强制删除 namespace
有时候 `kubectl` 删除 namespace 时，namespace 一直处于 **terminating**。这分两种场景：

+ 有时是 namespace 资源比较多，回收比较慢，这种情况只需要等待即可
+ 有时是卡死了，可以使用如下脚本删除

```bash
# 登录 k8s-master 节点, 查看 namespace 是否已经是 terminating 的状态了.
kubectl get ns | grep terminating | grep xxxx

# 导出 namespace 的 json 数据并删除 spe c中的所有内容
kubectl get namespace xxxx -o json > xxxx.json

# 新开一个 kubectl 的代理
kubectl proxy --port=8081

# 调用 finalize 的 api
curl -k -H "Content-Type: application/json" \
  -XPUT --data-binary @xxxx.json http://127.0.0.1:8081/api/v1/namespaces/xxxx/finalize
```

### 操作 Etcd
```bash
export ETCDCTL_API=3
alias etcdctl='etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key'
```

