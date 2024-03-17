---
id: k8s.crd
tags:
- CRD
- k8s
- kubernetes
title: "Kubernetes CRD \u5F00\u53D1"

---


# Kubernetes CRD 开发
参考文档：

- [https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#service-v1-core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#service-v1-core)

安装 Kubebuilder 脚手架：

- Go 环境
- Make
```bash
# 下载安装
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
chmod u+x kubebuilder && mv kubebuilder /usr/local/bin/

# 源码编译安装
git clone git@github.com:kubernetes-sigs/kubebuilder.git -b v3.9.0 --depth 1
make build 
mv bin/kubebuilder /usr/local/bin/kubebuilder

kubebuilder --help

mkdir kreutzer-operator
cd kreutzer-operator
go mod init github.com/leryn1122/kreutzer-operator/v2
kubebuilder init --domain leryn.github.io --owner leryn

kubebuilder create api --group kreutzer --version v1 --kind Pipeline
go mod tidy
make generate
```
