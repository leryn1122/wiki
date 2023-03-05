
<a name="xyGWN"></a>
# Helm
参考文档: 

- [Helm - 官网](https://docs.helm.sh/zh/docs/)
<a name="VMf5z"></a>
## 简介
Helm 是 k8s 的另外一个项目，相当于 linux 的 yum，但它不提供实际程序。Helm 仓库里面只有配置清单，镜像还是由镜像仓库来提供，比如 hub.docker.com。<br />Helm 提供了一个应用所需要的所有清单文件。比如对于一个 nginx，我们需要一个 deployment 的清单文件，一个 service 的清单文件，一个 hpa 的清单文件，把这三个文件打包到一起，就是一个应用程序的程序包，称之为 Chart。<br />Chart 是一个 Helm 程序包，实质只是一个模板，可以对这个模板进行赋值（value），形成我们自定义的清单文件。Helm 把 Kubernetes 资源打包到一个 Chart 中，保存到 Chart 仓库, 通过仓库来存储和分享。
<a name="c5t41"></a>
## 安装步骤
安装前需要有 k8s 服务，如果没有本地 k8s, 那么需要`--kube-apiserver localhost:8080`这样显式地指定 k8s 的服务器：
```bash
helm lint my-app --kube-apiserver localhost:8080
```
下载 helm 安装包，网络可能较慢需要代理：
```bash
wget https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz
tar -xf helm-*.tar.gz
mv $PWD/helm /usr/local/bin/helm
helm version
```
<a name="mg2k6"></a>
# Helm仓库
<a name="WYUDk"></a>
## 常用仓库
仓库不是越多越好，相反仓库过多可能每次更新仓库都很慢。
```bash
# helm 仓库
helm repo add bitnami         https://charts.bitnami.com/bitnami
helm repo add apphub          https://apphub.aliyuncs.com

# 更新Chart仓库
helm repo update

# 查看Chart仓库列表
helm repo list
```
<a name="JuyYW"></a>
## 安装插件
Helm 高于 v3.7.0，加入了 `helm push` 取代了原有 Chartmuseum 提供的 `helm-push` 插件。因此改为 `helm cm-push`。
```bash
# 安装插件
helm plugin install https://github.com/chartmuseum/helm-push.git
helm plugin install https://ghproxy.com/https://github.com/chartmuseum/helm-push.git

helm push sample-0.1.0.tgz devchartrepo
```
<a name="wXw7X"></a>
# 模板调试
```bash
# 检查chart包可能存在的问题
helm lint <CHART_NAME>

# 不安装chart, 让服务渲染模板后, 返回清单文件
helm install <RELEASE_NAME> --dry-run --debug <CHART_NAME>

# 或者
helm template --debug <RELEASE_NAME> <CHART_NAME>

helm get manifest <RELEASE_NAME>
```
<a name="AHScs"></a>
# Helm 语法
参考文档：

- [sprig 方法](https://masterminds.github.io/sprig/)


