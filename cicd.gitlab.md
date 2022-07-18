原有的 GitLab 和 Gitea 文档合并到这里.

![](https://s3.leryn.top/website/image/gitlab.svg#crop=0&crop=0&crop=1&crop=1&height=89&id=CkrvN&originHeight=887&originWidth=2500&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=250)

<a name="MAKhM"></a>
# GitLab 安装手册
<a name="yYh05"></a>
## Docker 安装
```bash
docker run \
  --detach=true \
  --publish=8501:80 \
  --publish=8502:443 \
  --publish=8503:22 \
  --restart always \
  --volume=/conf/gitlab:/etc/gitlab \
  --volume=/logs/gitlab:/var/log/gitlab \
  --volume=/data/gitlab:/var/opt/gitlab \
  --name=gitlab \
  --hostname=gitlab \
  gitlab/gitlab-ce:latest
```

![](./assets/1643290098256-ffd18248-4cdd-4720-a79c-69d116d29110.svg)
<a name="giIcc"></a>
# Gitea 安装手册
<a name="fC3oR"></a>
## Docker 安装

需要一个额外的数据库, 在设置向导中配置:

```bash
docker run \
  --detach=true \
  --publish=9022:22 \
  --publish=9080:3000 \
  --restart=always \
  --volume=/data/gitea:/data \
  --name=gitea \
  gitea/gitea:latest
```
