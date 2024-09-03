---
id: k8s.istio
tags:
- istio
- k8s
- kubernetes
title: "Istio - \u670D\u52A1\u7F51\u683C"

---
# Istio - 服务网格
参考文档:

+ [https://istio.io/latest/docs/setup/install/helm/](https://istio.io/latest/docs/setup/install/helm/)
+ [https://istio.io/latest/docs/setup/install/istioctl/](https://istio.io/latest/docs/setup/install/istioctl/)

### 安装 Istio
官方提供 `istioctl` 和 `Helm` 两种方式安装, 推荐使用 `Helm`.

```bash
# Configure the Helm repository
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

# Create namespace
kubectl create namespace istio-system

# Install base chart & istiod service
helm install istio-base istio/base -n istio-system
helm install istiod istio/istiod -n istio-system --wait

# Verfication
helm status istiod -n istio-system
```

### 开启 Istio
开启 Istio 的方式很简单, 在所处 namespace 上加上 `istio-injection=enabled`的 label, 再重启 Deployment, Statefulset, DaemonSet 即可.

```bash
# 
kubectl label namespace xxxx istio-injection=enabled --overwrite

# 这里重启你的 Deployment, Statefulset, DaemonSet

# 
kubectl get namespace -l istio-injection=enabled
```

