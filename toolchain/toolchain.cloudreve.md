---
id: toolchain.cloudreve
tags:
- software
- storage
title: Cloudreve

---
# Cloudreve
参考文档:

+ [配置文件 | 简体中文 | Cloudreve](https://docs.cloudreve.org/getting-started/config)
+ [https://hub.docker.com/r/xavierniu/cloudreve](https://hub.docker.com/r/xavierniu/cloudreve)

## 安装手册
### 获取 PUID 和 PGID
为什么要使用 PUID 和 PGID 参见[Understanding PUID and PGID](https://docs.linuxserver.io/general/understanding-puid-and-pgid)。假设当前登陆用户为 root，则执行`id root`就会得到类似于下面的一段代码：

```plain
uid=1000(root) gid=1001(root)
```

则在运行命令中的 PUID 填入 1000，PGID 填入 1001：

```bash
mkdir -p /data/cloudreve/uploads    # 上传目录
mkdir -p /data/cloudreve/config     # 配置文件夹
mkdir -p /data/cloudreve/db         # 数据库文件夹, 如
mkdir -p /data/cloudreve/avatar     # 头像文件夹
```

### 创建配置文件
创建配置文件(该配置文件针对 SQLite 数据库, 如需使用 MySQL 等数据库，请参考：[配置文件 | 简体中文 | Cloudreve](https://docs.cloudreve.org/getting-started/config)

```bash
vim /data/cloudreve/conf.ini
```

```properties
# conf.ini
[Database]
DBFile = /data/cloudreve/db/cloudreve.db
```

创建挂载物理卷：

```bash
mkdir -p /data/cloudreve/uploads
mkdir -p /data/cloudreve/config
mkdir -p /data/cloudreve/db
mkdir -p /data/cloudreve/avatar
```

### Docker 启动
```bash
docker run \
  --detach=true \
  --env=PUID=1000 \
  --env=PGID=1000 \
  --env=TZ="Asia/Shanghai" \
  --publish=5212:5212 \
  --restart=always \
  --volume=/data/cloudreve/uploads:/cloudreve/uploads \
  --volume=/data/cloudreve/config:/cloudreve/config \
  --volume=/data/cloudreve/db:/cloudreve/db \
  --volume=/data/cloudreve/avatar:/cloudreve/avatar \
  --name cloudreve \
  --hostname cloudreve \
  xavierniu/cloudreve:latest
```

查看初始密码：

```bash
docker logs cloudreve
```

