参考文档:

- [Registry API文档 - 官网](https://docs.docker.com/registry/spec/api/)
- [docker-registry-frontend - DockerHub](https://registry.hub.docker.com/r/konradkleine/docker-registry-frontend)


# DockerHub
DockerHub 是 Docker 官方提供的镜像仓库. 纵使 Docker 有种种问题, Kubernetes 也改用 containerd 作为默认 OCI 了, Dockerhub 生态使得 Docker 经久不衰.<br />DockerHub 任何人都可以注册免费账户, 并享有一个私有仓库, 以及每天 50 次免费 docker pull 请求. 基本够用, 在没有私有 registry 的情况下也足够开发者使用了. 不过由于下行宽带, 传输速度会比较慢, 这个时候镜像大小至关重要.<br />由于只有一个私有仓库, 所以我们规划一个映射, 使用 <image>-<tag> 拼接为最终的 tag: 

```bash
docker tag docker.leryn.top/<image>:<tag> leryn/app:<image>-<tag>
docker push leryn/app:<image>-<tag>
```


## Docker Resgisty (已过时)

Registry:

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

修改`/etc/docker/daemon.json`中配置的镜像源`registry-mirrors`, 不过当前没有二级域名的 SSL 证书, 所以在`insecure-registries`配置域名, 并重启 docker 服务:

```bash
vim /etc/docker/daemon.json
```
```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://harbor.leryn.top"
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


### Docker 命令行上传镜像

标记带有 registry 域名的 tag, 然后 push 即可:
```bash
docker tag openjdk:17-oracle docker.leryn.top/openjdk:17-oracle
docker push docker.leryn.top/openjdk:17-oracle
```


### HTTP API

最常用的两个 API:

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


## Docker Resgisty Frontend

参考文档:

- [konradkleine/docker-registry-frontend - DockerHub](https://registry.hub.docker.com/r/konradkleine/docker-registry-frontend)
- [kwk/docker-registry-frontend - GitHub](https://github.com/kwk/docker-registry-frontend)

这是一个开源的 docker registry 的 Web UI, 目前已经停止维护很久了, 它指具有只读的功能.

```bash
docker run \
  --detach=true \
  --env=ENV_DOCKER_REGISTRY_HOST=docker.leryn.top \
  --env=ENV_USE_SSL=yes \
  --env=ENV_MODE_BROWSE_ONLY=false \
  --publish=9543:443 \
  --restart=always \
  --volume=/etc/nginx/ssl/leryn.top/fullchain.cer:/etc/apache2/server.crt:ro \
  --volume=/etc/nginx/ssl/leryn.top/leryn.top.key:/etc/apache2/server.key:ro \
  --name=registry-ui \
  --hostname=registry-ui \
  konradkleine/docker-registry-frontend:v2

docker run \
  --detach=true \
  --env=ENV_DOCKER_REGISTRY_HOST=docker.leryn.top \
  --publish=9543:80 \
  --restart=always \
  --name=registry-ui \
  --hostname=registry-ui \
  konradkleine/docker-registry-frontend:v2
```

注意这里需要让`docker.leryn.top`支持跨域, 我使用 nginx 来实现.
