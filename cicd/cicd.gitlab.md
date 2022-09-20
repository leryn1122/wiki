原有的 GitLab 和 Gitea 文档合并到这里.
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
