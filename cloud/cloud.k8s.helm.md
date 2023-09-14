
# Helm Chart 介绍
参考文档：

- [Helm - 官网](https://docs.helm.sh/zh/docs/)

## 简介
Helm 是 Kubernetes 相关的另外一个项目，它提供了 Kubernetes 上应用包整合部署的一个解决方案。它已经成为了社区的事实标准，许多开源项目都会提供自己的 Chart 来部署应用。<br />Helm 仓库里面只有配置清单，镜像还是由镜像仓库来提供，比如 hub.docker.com。<br />Helm 提供了一个应用所需要的所有清单文件。比如部署一个 nginx，我们需要一份 deployment 的清单文件，一份service的清单文件，一个hpa的清单文件，把这三个文件打包到一起，就是一个应用程序的程序包，称之为Chart。<br />同类竞品还有：Kustomize 等。

## 安装步骤
安装前需要有 Kubernetes 服务，如果没有本地 Kubernetes，那么需要 `--kube-apiserver localhost:8080`这样显式地指定 Kubernetes 的服务器：
```bash
helm <command> --kube-apiserver localhost:8080
```
下载 Helm 安装包，网络可能较慢需要代理：
```bash
VERSION=v3.7.1
wget https://get.helm.sh/helm-${VERSION}-linux-amd64.tar.gz
tar -xf helm-*.tar.gz
mv helm /usr/local/bin
chmod u+x /usr/local/bin/helm
```
Helm 高于 `v3.7.0`，加入了 `helm push` 取代了原有 Chartmuseum 提供的 `helm-push` 插件。因此改为 `helm cm-push`。
```bash
# 安装插件
helm plugin install https://github.com/chartmuseum/helm-push.git

# Github 使用代理
helm plugin install https://ghproxy.com/https://github.com/chartmuseum/helm-push.git
```
Helm 常见命令：
```bash
# 添加仓库
helm repo add <REPO> <REPO_URL>

# 更新仓库
helm repo update

# 搜索仓库
helm search repo <CHART>

# 下载 Chart
helm pull repo <CHART> --version

# 新建 Chart
helm new <CHART>

# 更新依赖
helm dep update

# 检验 Chart
helm lint <CHART>

# 调试渲染 Chart
helm template --debug --dry-run <RELEASE> <CHART>

# 打包 Chart
helm package <DIR>

# 上传 Chart
helm cm-push <ARTIFACT> <REPO>

# 安装应用
helm instsall <RELEASE> <CHART> \
  -n <NAMESPACE> \
  -f values-custom.yaml \
  --create-namespace \
  --install
```

## Helm 仓库
Helm 可以用一下方式存储 Chart 用于分发：

- Chartmuseum：Helm 官方提供的软件
- Harbor：集成 Docker Registry 和 Chartmuseum 等等的开源软件
- Git 仓库（不推荐，经常引起版本冲突）

# Helm Chart 开发

## 目录结构
创建一个新的 Chart：
```bash
helm new helloworld
```
目录结构如下：
```bash
helloworld
├── charts                  # 依赖Chart都会下载在这里
├── Chart.yaml              # Chart项目元信息
├── templates               # 库/实际代码
│   └── deployment.yaml
└── values.yaml             # 部署时需要开放可配置的参数
```

## 语法
Helm 主要依赖于 Go template 语法，并加入了 [sprig 方法](https://masterminds.github.io/sprig/)。
