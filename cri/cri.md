
# 容器运行时
参考文档：

- [梳理一下 Windows 的 Hyper-V、Hypervisor](https://zhuanlan.zhihu.com/p/381969738)

广义的容器运行时有很多，包括但不限于以下：

- Docker
- Containerd
- Podman
- Kata-container
- NVIDIA Containerd （GPU 特供）

但是常提还是特指 Docker 和 Containerd<br />随着 OCI 的提出和厂商对隔离性的要求，除了用宿主机的进程隔离策略，现在有厂商提供基于虚拟机的容器方案，例如 Podman 在 MacOS 上提供基于 QEMU 的容器，Google 提出的 gVisor 等等。

## nerdctl & buildkit
nerdctl 旨在为 containerd 提供那些从 docker 中裁剪掉的功能。本身 runc 的 ctr 和 containerd 的 crictl 非常难使用，因为它们只提供了最基本的功能。nerdctl 的出现解决了这点。<br />如果需要完全替代 docker，nerdctl 还需要能构建镜像。但是 crictl 和 nerdctl 本身不支持构建，需要再额外安装一个 buildkitd 来作为 daemon 服务提供构建镜像的能力。<br />安装 nerdctl：
```bash
VERSION=1.7.3
wget https://github.com/containerd/nerdctl/releases/download/v${VERSION}/nerdctl-${VERSION}-linux-amd64.tar.gz

tar -xf nerdctl-${VERSION}-linux-amd64.tar.gz

sudo mv nerdctl /usr/local/bin/nerdctl
sudo chmod a+x /usr/local/bin/nerdctl
```
```bash
alias docker='nerdctl --namespace=k8s.io  --address=unix:///run/k3s/containerd/containerd.sock '
```
安装 buildkit 和 buildkitd：
```bash
VERSION=v0.12.5
wget https://github.com/moby/buildkit/releases/download/${VERSION}/buildkit-${VERSION}.linux-amd64.tar.gz

tar -xf buildkit-${VERSION}.linux-amd64.tar.gz

sudo mv bin/* /usr/local/bin/
sudo chmod a+x /usr/local/bin/build*
```
配置 systemd 文件：
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
Require=buildkit.socket
After=buildkit.socket
Documention=https://github.com/moby/buildkit
 
[Service]
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true
 
[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable  buildkitd
sudo systemctl restart buildkitd
sudo systemctl status  buildkitd
```
