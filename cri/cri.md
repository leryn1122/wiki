
# 容器运行时
广义的容器运行时有很多，包括但不限于以下：

- Docker
- Containerd
- Podman
- Kata-containerd
- NVIDIA Containerd （GPU 特供）

但是常提还是特指 Docker 和 Containerd


## nerdctl & buildkit
```bash
alias docker='nerdctl --namespace=k8s.io  --address=unix:///run/k3s/containerd/containerd.sock '
```
安装 buildkit 和 buildkitd：
```bash
wget https://github.com/moby/buildkit/releases/download/v0.12.5/buildkit-v0.12.5.linux-amd64.tar.gz

tar -xf buildkit-v0.12.5.linux-amd64.tar.gz

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
