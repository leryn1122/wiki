
![](https://s3.leryn.top/website/image/drone.svg#crop=0&crop=0&crop=1&crop=1&id=mIjNO&originHeight=150&originWidth=150&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

本流水线由于存在bug, 已经卸载
<a name="VZrAF"></a>
# drone 安装手册
参考文档:

- [drone - 官方文档](https://docs.drone.io)
- [drone 对接 gitee - 官方文档](https://docs.drone.io/server/provider/gitee/)
- [drone - 插件 - 官方文档](https://plugins.drone.io/)

不同的 git 产品对接方式各不相同, 请在官方文档中找到相应的对接方式, 这里只有 gitee.
<a name="Xjfkg"></a>
## Gitee
<a name="B1r7f"></a>
### OAuth 应用
在 Gitee 上创建 OAuth 应用程序<br />需要正确的应用主页和回调地址:

- 主页: [https://drone.leryn.top](https://drone.leryn.top)
- 回调: [https://drone.leryn.top/login](https://drone.leryn.top/login)

权限需要:

- user_info
- projects
- pull_requests
- notes
- hook
<a name="x9BwG"></a>
### 创建共享密钥
```bash
openssl rand -hex 16
```
```
2ff35a69ef0f5c60c781a8775e8db5a2
```
<a name="al0Kw"></a>
### Docker 安装 server

```bash
# Postgres
docker run \
  --detach=true \
  --env=DRONE_GITEE_CLIENT_ID=e59a0170ca8a5058aea083e5c9dfc3bc0640df1c0ca2f94d5cabc9994dc21d30 \
  --env=DRONE_GITEE_CLIENT_SECRET=a38039d4fb9d3424adfd93398672ad965264c97dabd00469e4370ab7354ef890 \
  --env=DRONE_RPC_SECRET=2ff35a69ef0f5c60c781a8775e8db5a2 \
  --env=DRONE_SERVER_HOST=drone.leryn.top \
  --env=DRONE_SERVER_PROTO=https \
  --env=DRONE_USER_CREATE=username:leryn,admin:true \
  --env=DRONE_DATABASE_DRIVER=postgres \
  --env=DRONE_DATABASE_DATASOURCE="postgres://drone:drone@123@121.196.30.39:5432/drone?sslmode=disable" \
  --publish=9180:80 \
  --publish=9443:443 \
  --restart=always \
  --name=drone \
  --hostname=drone \
  drone/drone:latest

# MySQL
docker run \
  --detach=true \
  --env=DRONE_GITEE_CLIENT_ID=e59a0170ca8a5058aea083e5c9dfc3bc0640df1c0ca2f94d5cabc9994dc21d30 \
  --env=DRONE_GITEE_CLIENT_SECRET=a38039d4fb9d3424adfd93398672ad965264c97dabd00469e4370ab7354ef890 \
  --env=DRONE_RPC_SECRET=2ff35a69ef0f5c60c781a8775e8db5a2 \
  --env=DRONE_SERVER_HOST=drone.leryn.top \
  --env=DRONE_SERVER_PROTO=https \
  --env=DRONE_USER_CREATE=username:leryn,admin:true \
  --env=DRONE_DATABASE_DRIVER=mysql \
  --env=DRONE_DATABASE_DATASOURCE="drone:3tDcHgbrFVkxBL0D@tcp(121.196.30.39:3306)/drone?parseTime=true" \
  --publish=9180:80 \
  --publish=9443:443 \
  --restart=always \
  --name=drone \
  --hostname=drone \
  drone/drone:latest
```

- DRONE_GITEE_CLIENT_ID<br />必需的字符串值提供您的 Gitee oauth 客户端 ID.
- DRONE_GITEE_CLIENT_SECRET<br />必需的字符串值提供您的 Gitee oauth 客户端密钥.
- DRONE_GITEE_SERVER<br />可选的 url 值提供 Gitee 服务器地址. 默认值为 gitee.com 服务器地址https://gitee.com.
- DRONE_GITEE_API_SERVER<br />可选字符串值提供 Gitee api 服务器地址. 默认值为 [https://gitee.com/api/v5](https://gitee.com/api/v5)
- DRONE_RPC_SECRET<br />必需的字符串值提供在上一步中生成的共享机密. 这用于验证服务器和运行程序之间的 rpc 连接. 必须为服务器和运行程序提供相同的秘密值.
- DRONE_SERVER_HOST<br />必需的字符串值提供您的外部主机名或 IP 地址. 如果使用 IP 地址, 您可以包含端口. 例如, drone.domain.com
- DRONE_SERVER_PROTO<br />必需的字符串值提供您的外部协议方案. 此值应设置为 http 或 https. 如果您配置 ssl 或 acme, 则此字段默认为 https

<a name="h6CTV"></a>
### Docker 安装 runner

Drone runner 是一个守护进程, 它在临时 docker 容器内执行管道步骤. 您可以安装单个 docker 运行器, 或在多台机器上安装 docker 运行器来创建您自己的构建集群.

```bash
docker run \
  --detach=true \
  --env=DRONE_RPC_PROTO=https \
  --env=DRONE_RPC_HOST=drone.leryn.top \
  --env=DRONE_RPC_SECRET=2ff35a69ef0f5c60c781a8775e8db5a2 \
  --env=DRONE_RUNNER_CAPACITY=2 \
  --env=DRONE_RUNNER_NAME=drone-runner \
  --env=DRONE_USER_CREATE=username:leryn,admin:true \
  --publish=9181:3000 \
  --restart=always \
  --name=drone-runner \
  --volume=/var/run/docker.sock:/var/run/docker.sock \
  drone/drone-runner-docker:latest
```

使用该 docker logs 命令查看日志并验证运行程序是否成功与 Drone 服务器建立连接

```bash
docker logs drone-runner
```

```
INFO[0000] starting the server                           addr=":3000"
INFO[0000] successfully pinged the remote server
INFO[0000] polling the remote server                     arch=amd64 capacity=2 endpoint="https://drone.leryn.top" kind=pipeline os=linux type=docker
```
<a name="nAvQD"></a>
## 流水线

搭建好 drone 后, 登录 OAuth2 认证. 点击`Activate Repository`激活仓库, 然后配置在仓库提交`.drone.yml`作为流水线配置即可启动流水线.
<a name="r0wwi"></a>
### 示例 .drone.yml
```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: build-on-commit-nightly
    image: plugins/docker
    settings:
      registry: docker.leryn.top
      repo: docker.leryn.top/leryn/image-name
      tags: [nightly]
    volumes:
      - name: docker-socket
        path: /var/run/docker.sock
    when:
      branch:
        exclude: [master, feature/*, stable]
      event:
        include: [cron, push, pull_request]
  - name: build-on-commit-latest
    image: plugins/docker
    settings:
      registry: docker.leryn.top
      repo: docker.leryn.top/leryn/image-name
      tags: latest
    volumes:
      - name: docker-socket
        path: /var/run/docker.sock
    when:
      branch:
        include: [master, feature/*, stable]
      event:
        include: [push, pull_request]
  - name: build-on-tag
    image: plugins/docker
    settings:
      registry: docker.leryn.top
      repo: docker.leryn.top/leryn/image-name
      tags: stable
    volumes:
      - name: docker-socket
        path: /var/run/docker.sock
    when:
      event:
        include: tag
        exclude: [cron, push, pull_request]
volumes:
  - name: docker-socket
    host:
      path: /var/run/docker.sock
```
<a name="dzosl"></a>
### 流水线优化
我们增加几步优化:

1. 把流水线所需要的镜像变成内网镜像(例如`plugins/docker`), 包括流水线本身和应用打包用到的镜像, 这样不会每次重复的向外网拉取镜像.
1. DNS: 由于我是单机节点, 直接配置`/etc/host`将域名指向到`localhost`.
1. 挂载`/var/run/docker.sock`到内部, 这需要是 Drone Runner 挂载这个 socket 并且将仓库设置`Trusted`. 这可以使与 Docker 传输加快.
1. 优化 Dockerfile: 利用 Dockerfile 缓存.
