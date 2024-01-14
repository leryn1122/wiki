
# Kind - Kubernetes in Docker
Kind (Kubernetes IN Docker) 是一个用于快速创建测试的 Kubernetes，仅仅需要安装 docker 或者 podman 即可。

## 安装步骤
前置条件

- 需要额外安装 docker 或者 podman

只需要手动安装二进制即可：
```bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-arm64

chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
