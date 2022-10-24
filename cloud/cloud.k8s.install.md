<a name="kk8cA"></a>
# Kubernetes 安装手册
<a name="m2I7s"></a>
## 前置准备
前置准备：

- 安装前置依赖
- 安装 Docker，或者其他容器运行时

安装前置依赖：
```bash
sudo apt update && sudo apt install -y \
  apt-transport-https ca-certificates curl gnupg2 software-properties-common
```
<a name="RSOJA"></a>
## 安装应用
添加 Kubernetes 源：
```bash
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6A030B21BA07F4FB
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys FEEA9169307EA071
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8B57C5C2836F4BEB
sudo apt update

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
#deb https://apt.kubernetes.io/ kubernetes-xenial main
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
deb https://mirrors.cloud.tencent.com/kubernetes/apt kubernetes-xenial main
EOF
```
安装 Kubelet：
```bash
KUBE_VERSION=1.18.16-00
sudo apt install -y kubelet=${KUBE_VERSION} kubeadm=${KUBE_VERSION} kubectl=${KUBE_VERSION}
sudo apt-mark hold kubelet kubeadm kubectl docker-ce docker-ce-cli containerd
```
修改网络配置，开启 Linux 内核中的 iptables 模块，Kubernetes 网络默认使用 iptables 来实现 Service 功能：
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```bash
sudo sysctl --system
modprobe br_netfilter
lsmod | grep br_netfilter
```
关闭 swap 分区。如果开启 swap 分区，当OOM时内存被交换到磁盘上时，这个节点可能会挂住，而且没有任何报错信息提示，甚至无法使用物理终端访问，最后只能硬重启整个节点。所以请关闭 swap 分区。
```bash
swapoff -a
edit /etc/fstab
# 注释掉 swap 分区
```
<a name="lYKDx"></a>
## 高可用集群安装
如果一个 Kubernetes 要满足高可用，那么建议保证集群有：

- 至少 3 个 master 节点
- 至少 3 个 worker 节点
- 至少 2 个HAProxy 和至少 2 个KeepAlived，建议和节点分离部署
- 一个额外的空闲 IP
<a name="YBZKZ"></a>
### HAProxy 安装
```bash
sudo apt update && sudo apt install -y \
  haproxy
```
修改配置文件：
```bash
vim /etc/haproxy/haproxy.cfg
```
在默认配置后面追加 master:6443, worker:80/443 的高可用：
```
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
<a name="B8MYl"></a>
### KeepAlived 安装
KeepAlived 用于占用 VIP（虚拟IP），不需要实际的服务器，但要占用一个空闲 IP。KeepAlived 会轮流占用这个 VIP，如果当前 KeepAlived 宕机，那么 VIP 会漂移到另一个 KeepAlived 之上，实现高可用。
```bash
sudo apt update && sudo apt install -y \
  keepalived
```
更改配置文件：
```bash
vim /etc/keepalived/keepalived.conf
```
```
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
systemctl enable keepalived
systemctl restart keepalived
systemctl status keepalived
```
然后我们初始化集群之前要将 Controll Plane 域名解析指向这个 VIP 的地址。
<a name="LWZVp"></a>
## 初始化集群
初始化集群：如果执行成功，控制台会打印 `kubeadm join` 命令，依次在对应的节点上运行：

- 第一条是加入 master 节点使用的
- 第二条是加入 worker 节点使用的
```bash
kubeadm init \
  --control-plane-endpoint xxx-k8s-master.domain.com \
  --image-repository k8s.gcr.io/google_containers \
  --upload-certs \
  --pod-network-cidr=172.24.0.0/16 \
  --kubernetes-version 1.18.16
```
如果报错了可以随时清除设置并重新初始化：
```bash
kubeadm reset
systemctl daemon-reload
systemctl restart kubelet
```
如果超过 24 小时，Token 会自动过期，用以下命令生成新的 Token：
```bash
kubeadm token create --print-join-command
```
初始化集群后，会产生静态 Pod。这些 Pod 的 Manifest 文件会存在于指定目录下，标准的 Kubernetes 是 `/etc/kubernetes/manifests`。Kubernetes 始终保证这些 Pod 会存在，如果它们被误删除，过一段时间后也将重新生成。Kubernetes 内部组件的配置在这里修改：
```bash
# APIServer 开启8080端口 所有master节点都修改
vim /etc/kubernetes/manifests/kube-apiserver.yaml
# --insecure-port=8080 

# kubectl get cs 状态是 unhealthy, 删除下面文件的配置 port=0 这一行
vim /etc/kubernetes/manifests/kube-scheduler.yaml
vim /etc/kubernetes/manifests/kube-controller-manager.yaml

#  集群开放 30000 以下的端口, 需要手动修改 kubelet 的配置文件
vim /etc/kubernetes/manifests/kube-apiserver.yaml 
# 添加内容: --service-node-port-range=1-65535
```
安装 CNI 网络插件：

- 虽然不安装网络插件无法让 Kubernetes 集群通讯，但是 Kubernetes 官方认为 CNI 不是它的范围，也没有提供默认的网络设施。
- CNI 网络插件实现非常多：Weave，Flannel，Calico 等等。这里安装 Weave，因为我司用的 Weave，但它不是目前最好的实现。
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
安装完成后，验证集群：
```bash
# 查看节点状态, 全是 ready 就对了
kubectl get nodes

# 查看集群健康状况
kubectl get cs
```
