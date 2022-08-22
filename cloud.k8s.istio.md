<a name="wBf34"></a>
# Istio 安装步骤
<a name="iUol9"></a>
## 二进制安装

```bash
wget https://storage.googleapis.com/istio-release/releases/1.10.3/istio-1.10.3-linux-amd64.tar.gz
tar -xf istio-1.10.3-linux-amd64.tar.gz
mv istio-1.10.3 /opt/module/istio-1.10.3
```

```bash
#!/usr/bin/env bash
export ISTIO_VERSION=1.10.3
export ISTIO_HOME=/opt/module/istio-${ISTIO_VERSION}
export PATH=${ISTIO_HOME}/bin:$PATH
```

```bash
istioctl version
```

```bash
# istio相关镜像 和 prometheus 镜像的仓库, 强制安装.
istioctl manifest apply \
  --set hub=dockerhub.azk8s.cn/istio \
  --set values.prometheus.hub=dockerhub.azk8s.cn/prom \
  --force
```
