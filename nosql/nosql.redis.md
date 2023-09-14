
# Redis 安装手册
参考文档：

- [Redis - 官网](https://redis.io/)
- [Redis 集群模式介绍](http://www.zzvips.com/article/142376.html)

## 包管理器
直接使用默认源 apt 不能下载到最新的 redis，先更新 apt 的源：<br />**方法一**<br />将如下链接添加到`/etc/apt/sources.list`中：
```
deb http://ppa.launchpad.net/rwky/redis/ubuntu trusty main
deb-src http://ppa.launchpad.net/rwky/redis/ubuntu trusty main
```
**方法二**
```bash
sudo add-apt-repository -y ppa:rwky/redis
```
然后直接下载`redis`：
```bash
sudo apt update
sudo apt install -y redis-server   # redis 服务端
sudo apt install -y redis-sentinel # redis 哨兵
```

## Docker 安装
注意一点：如果挂载了`redis.conf`，那么必须设为`daemonize no`，否则无法启动。原因很简单，如果使用 Daemon 守护进程，那么 1 号进程退出了，Docker 会自动停止，所以得阻塞住 1 号进程。
```bash
docker run \
  --detach=true \
  --publish=6379:6379 \
  --volume=/conf/redis/redis.conf:/usr/local/etc/redis/redis.conf \
  --volume=/data/redis:/data \
  --name=redis \
  --hostname=redis \
  redis:6.2.8-alpine \
  redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

## 启动与验证
验证 Redis 是否安装成功，使用命令查看版本，如果可以正确显示版本信息则安装成功。
```bash
docker run -it --rm redis:6.2.8-alpine redis-server -v 
```
```
Redis server v=6.2.8 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=646e2341a1052373
```
启动或停止服务，如果修改了配置文件，需要显式地指定配置文件。
```bash
# 启动Redis
redis-server redis.conf
# 停止Redis
redis-cli shutdown
```
使用`src/redis-cli`进入命令行界面，如果可以正常操作则安装成功。
```bash
redis-cli
```
使用 Redis API 测试键值对是否能够正常运作。
```bash
# 验证密码
127.0.0.1:6379> auth 密码
# 测试键值对
127.0.0.1:6379> set foo "John Doe"
127.0.0.1:6379> get foo
127.0.0.1:6379> del foo
127.0.0.1:6379> exit
```

## 高可用集群
Redis 高可用集群至少需要 6 台服务器：其中三台形成一主两从的主从结构，另外三台作为哨兵监听这三台集群，形成高可用。如果主节点宕机，哨兵会进行投票选举出一个从节点晋升为新的主节点。<br />暂时以伪分布式（单机多容器、多进程）搭建高可用集群，一个集群模拟一个节点。
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
| 节点名 | 角色 | 端口 |
| --- | --- | --- |
| redis-01 | master | 6379 |
| redis-02 | master | 6378 |
| redis-03 | master | 6377 |
| redis-sentinel-01 | sentinel | 26379 |
| redis-sentinel-02 | sentinel | 26378 |
| redis-sentinel-03 | sentinel | 26377 |

