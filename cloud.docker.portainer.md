![](https://s3.leryn.top/website/image/portainer.svg#crop=0&crop=0&crop=1&crop=1&height=201&id=xQbLD&originHeight=731&originWidth=2500&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=687)
<a name="lRq53"></a>
# Portainer 安装手册
参考文档:

- [Portainer - 官方文档](https://docs.portainer.io/)
- [Portainer - DockerHub](https://hub.docker.com/r/portainer/portainer-ce)

Portainer 是一个管理 docker 容器的 UI 界面. 尽管现在我们有了 Kubernetes 容器编排, 但需要一些前置的基础设施. 对于小型的开发机, Portainer 仍然是一个不错的选择.

Portainer 社区版免费试用, 注册账号使用商业版可以免费获得 5 个节点, 这里仍然选用社区版.

<a name="IeksZ"></a>
## Docker 安装

这里安装后我们会以远程的方式连接服务器上的 Docker. 如果需要直接管理本地 Docker, 需要挂载 Docker 的`/var/dock/docker.sock`到容器内.

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
