
# Docker 周边生态

## Docker
十分推荐使用 Ubuntu 系统安装 Docker，而不是 CentOS。这基于以下原因：

- Linux 内核会为 containerd 让路并提供相应的支持，而 CentOS 的发行版的更新速度完全跟不上变化速度
- 相对比较多的开发者使用 Ubuntu 来作为开发机或开发环境的构建

### 包管理器安装
```bash
sudo apt update && sudo apt install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"

sudo apt update && sudo apt install -y \
  containerd.io docker-ce docker-ce-cli

cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "features": {
    "buildkit": true
  }
}
EOF
```

### Docker API

- [Develop with Docker Engine API](https://docs.docker.com/engine/api/)
- [Develop with Docker Engine SDKs](https://docs.docker.com/engine/api/sdk/#unofficial-libraries)

#### 命令行
查看 docker 存储空间
```bash
# 查看空间
docker system df

# 命令可以进一步查看空间占用细节, 以确定是哪个镜像、容器或本地卷占用过高空间
docker system df -v
```
清理空间

- 已停止的容器：指所有已被容器（包括 stop 的）关联的镜像，也就是 `docker ps -a` 所看到的所有容器对应的 image
- 未被引用的镜像：没有被分配或使用在容器中的镜像
- 未被任何容器使用的卷
- 未被任何容器所关联的网络
- 所有悬空的镜像：未配置任何 Tag（也就是无法被引用）的镜像。通常是由于镜像编译过程中未指定 `-t` 参数配置 Tag 导致
```bash
#
docker system prune
# 一并清除所有未被使用的镜像和悬空镜像
docker system prune -a
# 用以强制删除, 不提示信息
docker system prune -f
# 删除悬空的镜像
docker image prune
# 删除无用的容器
docker container prune
#
docker container prune --filter "until=24h"
# 删除无用的卷
docker volume prune
# 删除无用的网络
docker network prune
# 
docker builder prune
```
手动清除：<br />对于悬空镜像和未使用镜像可以使用手动进行个别删除：
```bash
# 删除所有悬空镜像, 不删除未使用镜像:
docker rmi $(docker images -f "dangling=true" -q)

# 删除所有未使用镜像和悬空镜像
docker rmi $(docker images -q)

# 清除一些不使用的卷
docker volume rm $(docker volume ls -qf dangling=true)

# 删除所有已退出的容器：
docker rm -v $(docker ps -aq -f status=exited)

# 删除所有状态为 dead 的容器
docker rm -v $(docker ps -aq -f status=dead)
```
批量操作
```bash
# 更新所有latest镜像
docker images --format "{{.Repository}}:{{.Tag}}" | grep ':latest' | xargs -L1 docker pull

# 更新所有镜像
docker images --format "{{.Repository}}:{{.Tag}}" | xargs -L1 docker pull

# 强制删除空镜像
docker images | awk '$1=="<none>"||$2=="<none>"{print $3}' | xargs -r docker rmi --force
```

#### HTTP 接口
:::danger
**请不要在未加 TLS 证书的情况下，向外部开放 TCP 端口。**
:::
如果未加 TLS 证书，那么任何人都可以通过 HTTP 调用你的 Docker，可以在你的服务器上启动镜像，通过挂载本地卷或者容器逃逸的方式，获得宿主机的 root 权限。

在 systemctl 的文件中配置端口，追加一句 `-H tcp://0.0.0.0:12375`（目前我的 docker 服务尚未配置 TLS 安全证书，仅用于测试）：
```bash
sudo vim /usr/lib/systemd/system/docker.service
```
```bash
# 由于公网上默认端口2375比较容易收到攻击, 换个12375端口
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:12375
```
远程构建镜像：需要将本地项目打包成 tar 包，再使用 HTTP 工具发送到远端。tar 包解压出的文件夹将作为 docker 构建的上下文路径，甚至可以将这个方法包装为前端项目的构建脚本，而不用流水线。
```bash
curl -XPOST 'http://xxx.xxx.xxx.xxx:2375/build?t=image:tag' \
     -H 'application/x-tar' \
     --data-binary @app.tar.gz
```


## Registry
参考文档：

- [Registry](https://docs.docker.com/registry/spec/api/)
- [https://registry.hub.docker.com/r/konradkleine/docker-registry-frontend](https://registry.hub.docker.com/r/konradkleine/docker-registry-frontend)

### DockerHub 和其他公开源
DockerHub 是 Docker 官方提供的镜像仓库。纵使 Docker 有种种问题，Kubernetes 也改用 containerd 作为默认 OCI 了，但是 Dockerhub 丰富生态使得 Docker 经久不衰。其他公开源：

- hub.docker.com
- quay.io
- gcr.io
- ghcr.io
- registry.cn-hangzhou.aliyuncs.com
- docker.mirrors.ustc.edu.cn
- registry.docker-cn.com

DockerHub 任何人都可以注册免费账户，并享有一个私有仓库，以及每天 50 次免费 docker pull 请求基本够用。在没有私有 registry 的情况下也足够开发者使用了。不过由于下行宽带，传输速度会比较慢，这个时候镜像大小至关重要。<br />由于只有一个私有仓库，所以我们规划一个映射，使用 `<image>-<tag>` 拼接为最终的 tag：
```bash
docker tag docker.mydomain.com/<image>:<tag> leryn/app:<image>-<tag>
docker push leryn/app:<image>-<tag>
```

### ~~Docker Resgisty 安装手册（已过时）~~
```bash
docker run \
  --detach=true \
  --publish=5000:5000 \
  --restart=always \
  --volume=/data/registry:/var/lib/registry \
  --volume=/conf/registry:/etc/docker/registry \
  --name=registry \
  --hostname=registry \
  registry:latest
```
修改`/etc/docker/daemon.json`中配置的镜像源`registry-mirrors`，不过当前没有二级域名的 SSL 证书，所以在`insecure-registries`配置域名，并重启 docker 服务：
```bash
vim /etc/docker/daemon.json
```
```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "host": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "features": {
    "buildkit": true
  }
}
```

### HTTP API
最常用的两个 API：
```http
GET /v2/_catalog
```
```json
{
  "repositories": [
    "openjdk"
  ]
}
```
```http
GET /v2/openjdk/tags/list
```
```json
{
  "name": "openjdk",
  "tags": [
    "17-oracle",
    "17-slim"
  ]
}
```

## ~~Docker Resgisty Frontend（已过时）~~
参考文档：

- [https://registry.hub.docker.com/r/konradkleine/docker-registry-frontend](https://registry.hub.docker.com/r/konradkleine/docker-registry-frontend)
- [https://github.com/kwk/docker-registry-frontend](https://github.com/kwk/docker-registry-frontend)

这是一个开源的 docker registry 的 Web UI，目前已经停止维护很久了，它指具有只读的功能。
```bash
docker run \
  --detach=true \
  --env=ENV_DOCKER_REGISTRY_HOST=docker.mydomain.com \
  --env=ENV_USE_SSL=yes \
  --env=ENV_MODE_BROWSE_ONLY=false \
  --publish=9543:443 \
  --restart=always \
  --volume=/etc/nginx/ssl/leryn.top/fullchain.cer:/etc/apache2/server.crt:ro \
  --volume=/etc/nginx/ssl/leryn.top/mydomain.com.key:/etc/apache2/server.key:ro \
  --name=registry-ui \
  --hostname=registry-ui \
  konradkleine/docker-registry-frontend:v2

docker run \
  --detach=true \
  --env=ENV_DOCKER_REGISTRY_HOST=docker.mydomain.com \
  --publish=9543:80 \
  --restart=always \
  --name=registry-ui \
  --hostname=registry-ui \
  konradkleine/docker-registry-frontend:v2
```
注意这里需要让`docker.mydomain.com`支持跨域，我使用 nginx 来实现。

## Portainer
参考文档：

- [Welcome - Portainer Documentation](https://docs.portainer.io/)
- [https://hub.docker.com/r/portainer/portainer-ce](https://hub.docker.com/r/portainer/portainer-ce)

Portainer 是一个管理 docker 容器的 UI 界面。尽管现在我们有了 Kubernetes 容器编排，但需要一些前置的基础设施。对于小型的开发机，Portainer 仍然是一个不错的选择。Portainer 社区版免费试用，注册账号使用商业版可以免费获得 5 个节点，这里仍然选用社区版。<br />这里安装后我们会以远程的方式连接服务器上的 Docker。如果需要直接管理本地 Docker，需要挂载 Docker 的`/var/dock/docker.sock`到容器内。
```bash
docker run \
  --detach=true \
  --publish=8000:8000  \
  --publish=9000:9000  \
  --restart=always \
  --volume=/data/portainer:/data  \
  --volume=/var/run/docker.sock:/var/run/docker.sock \
  --name=portainer \
  --hostname=portainer \
  portainer/portainer-ce:latest
```
