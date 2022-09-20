
参考文档:

- [K3S 安装文档 - 官方](https://docs.rancher.cn/docs/k3s/installation/install-options/_index)

K3S 是 Rancher 官方提出的轻量级 Kubernetes, 阉割或降级了很多 Kubernetes 的配置, 例如用 SQLite 来替换 Etcd, 但它仍然满足 Kubernetes 的所有 API. 性能上, 资源占用比 Minikube 更小, 是 Kubernetes 开发机最合适的产品实现.

![](./../assets/1645099177426-d5558c49-8ada-47c2-938f-2841e284111d.svg)
<a name="fm1SD"></a>
## 安装步骤

- 这里我们加上参数`INSTALL_K3S_EXEC="--no-deploy traefik"`. 之后我们会用 ingress-nginx-controller 来代替 traefik. 如果你更熟悉 traefik, 可以继续使用, 但本人更加熟悉 nginx.
- 默认使用 containerd 作为容器运行时，如果需要使用 docker 作为容器运行时, 加上 `--docker` 参数

```bash
# 一键安装k3s
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--no-deploy traefik" sh -s - --docker

# 国内加速使用
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | \
  INSTALL_K3S_MIRROR=cn \
  INSTALL_K3S_EXEC="--no-deploy traefik " sh -s - --docker
```

如果失败了, `journactl -xe` 提示:

> kubelet cgroup driver: \"cgroupfs\" is different from docker cgroup driver: \"systemd\""


修改 `/etc/docker/daemon.json`中的 cgroup driver:

```bash
{
...

  "exec-opts": ["native.cgroupdriver=cgroupfs"],

...
}
```

```bash
# 查看版本
sudo k3s kubectl version

# 查看node
sudo k3s kubectl get nodes

# 查看pods
sudo k3s kubectl get pods
```

其实安装之后 `/usr/local/bin` 下有了这几个命令:
```bash
/usr/local/bin/k3s
/usr/local/bin/k3s-killall.sh
/usr/local/bin/k3s-uninstall.sh
/usr/local/bin/k9s
/usr/local/bin/kubectl -> k3s
```
(可选) 配置一下别名调用 k3s 的 API.

(可选) kubeconfig 会安装在 `/etc/rancher/k3s/k3s.yaml`:

```bash
cat <<EOF | sudo tee /etc/profile.d/env-k3s.sh
#!/usr/bin/env bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
alias kubectl='sudo k3s kubectl '
EOF
```

<a name="FnBl8"></a>
## 卸载 K3S

如果您使用安装脚本安装了 

k3s, 那么在安装过程中会生成一个卸载 k3s 的脚本. 卸载 k3s 会删除集群数据和所有脚本. 要使用不同的安装选项重新启动集群, 请使用不同的标志重新运行安装脚本.<br />要从 server 节点卸载 k3s, 请运行:

```bash
/usr/local/bin/k3s-uninstall.sh
```

要从 agent 节点卸载 k3s, 请运行:

```bash
/usr/local/bin/k3s-agent-uninstall.sh
```

<a name="yFW58"></a>
## Helm支持

正常安装 K3S 无法直接使用 Helm:

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```
