<a name="bnJCx"></a>
# Redis 安装手册
![redis.svg](./assets/1660030223577-eb896851-733c-4647-804d-6e3e50592ef1.svg)

Reference:

- [Redis - 官网](https://redis.io/)
- [Redis 集群模式介绍](http://www.zzvips.com/article/142376.html)
<a name="bq351"></a>
## 包管理器
直接 apt 不能下载到最新的 redis, 先更新 apt 的源:

**方法一**

将如下链接添加到`/etc/apt/sources.list`中
```
deb http://ppa.launchpad.net/rwky/redis/ubuntu trusty main
deb-src http://ppa.launchpad.net/rwky/redis/ubuntu trusty main
```


**方法二**

```bash
sudo add-apt-repository -y ppa:rwky/redis
```

然后直接下载`redis`

```bash
sudo apt update
sudo apt install -y redis-server   # redis 服务端
sudo apt install -y redis-sentinel # redis 哨兵
```
<a name="XBSk4"></a>
## Docker 安装

注意一点: 如果挂载了`redis.conf`, 那么必须设为`daemonize no`, 否则无<br />法启动.

```bash
docker run \
  --detach=true \
  --publish=6379:6379 \
  --volume=/conf/redis/redis.conf:/etc/redis/redis.conf \
  --volume=/data/redis:/data \
  --name=redis \
  --hostname=redis \
  redis:6.2.5-alpine \
  redis-server /etc/redis/redis.conf --appendonly yes
```
<a name="f3OFb"></a>
## 二进制安装
<a name="ZFZeQ"></a>
### 前置准备

部署准备, 安装前需要准备如下材料:

- Redis 安装包: 

```bash
wget https://download.redis.io/releases/redis-6.2.1.tar.gz
```

根据不同集群模式选择:

- 主从模式需要至少 2 台;
- 哨兵模式需要至少 3 台;
- 集群模式需要至少 6 台.

<a name="qSPAn"></a>
### 安装步骤

配置环境变量, 并使其生效.

```bash
# Set Redis environment.
export REDIS_HOME=/opt/module/redis-6.2.1
export PATH=${PATH}:${REDIS_HOME}/src
```

解压安装包到指定路径:

```bash
tar -xvzf redis-6.2.1.tar.gz
mv redis-6.2.1 ${REDIS_HOME}
cd ${REDIS_HOME}
```

编译 C 语言文件:

```bash
make
```

修改`redis.conf`配置文件:

```bash
cp -ar redis.conf redis.conf.bak
vim redis.conf
```
```bash
# 默认绑定本机IP
bind 127.0.0.1 -::1

# 改为daemon模式启动
daemonize yes

# pid文件路径
pidfile /opt/data/redis/redis_6379.pid

# 数据库持久化文件
dir /opt/data/redis/

# 设置密码 将明文写在配置文件中
requirepass 密码明文
```
<a name="mqs63"></a>
### 启动与验证

验证 Redis 是否安装成功, 使用命令查看版本, 如果可以正确显示版本信息则安装成功.

```bash
redis-server -v
```
```
Redis server v=6.2.1 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=24aa46dd28217799
```

启动或停止服务, 如果修改了配置文件, 需要显式地指定配置文件.

```bash
# 启动Redis
redis-server redis.conf
# 停止Redis
redis-cli shutdown
```

使用`src/redis-cli`进入命令行界面, 如果可以正常操作则安装成功.

```bash
redis-cli
```

使用 Redis API 测试键值对是否能够正常运作.

```bash
# 验证密码
127.0.0.1:6379> auth 密码
# 测试键值对
127.0.0.1:6379> set foo "John Doe"
127.0.0.1:6379> get foo
127.0.0.1:6379> del foo
127.0.0.1:6379> exit
```
<a name="hbvFs"></a>
### 高可用集群

Redis 高可用集群至少需要 6 台服务器: 其中三台形成一主两从的主从结构, 另外三台作为哨兵监听这三台集群, 形成高可用. 如果主节点宕机, 哨兵会进行投票选举出一个从节点晋升为新的主节点.<br />暂时以伪分布式(单机多进程)搭建高可用集群, 一个集群模拟一个节点.

```bash
# 主从配置文件
cp -ar redis.conf conf/redis-01.conf  # master
cp -ar redis.conf conf/redis-02.conf  # slave
cp -ar redis.conf conf/redis-03.conf  # slave
# 哨兵配置文件
cp -ar sentinel.conf conf/sentinel-01.conf   # sentinel
cp -ar sentinel.conf conf/sentinel-02.conf
cp -ar sentinel.conf conf/sentinel-03.conf
```

| 节点名 |  | 端口 |
| --- | --- | --- |
| redis-01 |  | 6379 |
| redis-02 |  | 6378 |
| redis-03 |  | 6377 |
| redis-sentinel-01 |  | 26379 |
| redis-sentinel-02 |  | 26378 |
| redis-sentinel-03 |  | 26377 |


需要改配置文件, 单机版可以用端口号来标记.

```bash
# master 6379
# 绑定主机IP
bind 0.0.0.0
# PID文件
pidfile /opt/data/redis/pid/redis_6379.pid
# 日志文件
logfile "/opt/data/redis/logs/redis_6379.log"
# 数据库文件
dbfilename dump_6379.rdb
# 工作目录
dir /opt/data/redis/
# 开启aof模式
appendonly yes
# 一旦master宕机变为salve重新上线, 需要也需要主机密码
masterauth Redis123gogogo
```

从机配置:

```bash
# slave 6378, 6377
# 其他参考master配置
# slave授予master密码
masterauth Redis123gogogo
# 标记master
slaveof 121.196.30.39 6379
```

哨兵配置:

```bash
# sentinel 26379, 26378, 26377
# PID文件
pidfile "/opt/data/redis/pid/redis-sentinel_26379.pid"
# 日志文件
logfile "/opt/data/redis/logs/redis-sentinel_26379.log"
# 工作目录 哨兵似乎没有什么工作文件
dir "/tmp/sentinel-01"
# 标记监听master
sentinel monitor redis-01 121.196.30.39 6379 2
# 哨兵授予master密码
sentinel auth-pass redis-01 Redis123gogogo
```

封装一个集群启动脚本:

```bash
cat <<\EOF > start-redis-cluster.sh
#!/usr/bin/env bash
this="${REDIS_HOME}/src"
cd "${REDIS_HOME}" || exit 1
"${this}/redis-server"    "${this}/../conf/redis-01.conf"
"${this}/redis-server"    "${this}/../conf/redis-02.conf"
"${this}/redis-server"    "${this}/../conf/redis-03.conf"
"${this}/redis-sentinel"  "${this}/../conf/sentinel-01.conf"
"${this}/redis-sentinel"  "${this}/../conf/sentinel-02.conf"
"${this}/redis-sentinel"  "${this}/../conf/sentinel-03.conf"
EOF
```

启动集群:

```bash
# 启动Redis集群
./start-redis-cluster.sh
# 检查进程
ps -ef | grep redis | grep -v grep
```
<a name="CTjUr"></a>
### 高可用测试

启动集群后登录 master 输入键值对:

```bash
redis-cli -a Redis123gogogo -p 6379
127.0.0.1:6379> set foo "Hello world"
127.0.0.1:6379> get foo
```

在 slave 查询键值对:

```bash
redis-cli -a Redis123gogogo -p 6378
127.0.0.1:6378> set foo "Hello world"
127.0.0.1:6378> get foo
```

在 sentinel 上查询哨兵信息:

```bash
redis-cli -a Redis123gogogo -p 26379
127.0.0.1:26379> info sentinel
```
