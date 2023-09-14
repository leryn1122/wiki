
# Doris 安装手册
参考文档：

- [源码编译 - Apache Doris 中文文档](https://doris.apache.org/zh-CN/installing/compilation.html)
- [安装与部署 - Apache Doris 中文文档](https://doris.apache.org/zh-CN/installing/install-deploy.html)
- [集群升级 - Apache Doris 中文文档](https://doris.apache.org/zh-CN/installing/upgrade.html)
- [Doris ver. 1.0.0-rc4 下载地址](https://jiafeng2022.oss-cn-beijing.aliyuncs.com/doris-1.0.0.tar.gz?Expires=1648430085&OSSAccessKeyId=TMP.3KdkAydTCeb9cUUQ1LHqunP2oucQ2T1dbyGMwqxkij2SrRWNVUE1HYqz1omC1DpMVzqefLcSRxLr8mDigGgYxMKtLESxwo&Signature=Y2TT3%2BhnNBVfx9HCAttPa5bkin0%3D)

## 源码编译

### 使用 Docker 开发镜像编译（推荐）
针对不同的 Doris 版本，需要下载对应的镜像版本。从 Apache Doris 0.15 版本起，后续镜像版本号将与 Doris 版本号统一。比如可以使用 `apache/incubator-doris:build-env-for-0.15.0` 来编译 0.15.0 版本。<br />`apache/incubator-doris:build-env-ldb-toolchain-latest` 用于编译最新主干版本代码，会随主干版本不断更新。可以查看 docker/README.md 中的更新时间。<br />从 build-env-1.3.1 的 docker 镜像起，同时包含了 OpenJDK 8 和 OpenJDK 11，并且默认使用 OpenJDK 11 编译。请确保编译使用的 JDK 版本和运行时使用的 JDK 版本一致，否则会导致非预期的运行错误。你可以使用在进入编译镜像的容器后，使用以下命令切换默认 JDK 版本：<br />**切换到 JDK 8：**
```bash
alternatives --set java java-1.8.0-openjdk.x86_64
alternatives --set javac java-1.8.0-openjdk.x86_64
export JAVA_HOME=/usr/lib/jvm/java-1.8.0
```
**切换到 JDK 11：**
```bash
alternatives --set java java-11-openjdk.x86_64
alternatives --set javac java-11-openjdk.x86_64
export JAVA_HOME=/usr/lib/jvm/java-11
```

### 运行镜像
建议以挂载本地 Doris 源码目录的方式运行镜像，这样编译的产出二进制文件会存储在宿主机中，不会因为镜像退出而消失。同时，建议同时将镜像中 maven 的 .m2 目录挂载到宿主机目录，以防止每次启动镜像编译时，重复下载 maven 的依赖库。
```bash
VERSION=0.15.0
docker run -itd \
  -v $(cd ~; pwd)/.m2:/root/.m2 \
  -v $(cd ~; pwd)/incubator-doris-DORIS-$VERSION-release/:/root/incubator-doris-DORIS-$VERSION-release/ \
  apache/incubator-doris:build-env-for-$VERSION
```
下载源码启动镜像后，你应该已经处于容器内。可以通过以下命令下载 Doris 源码（已挂载本地源码目录则不用）：
```bash
VERSION=0.15.0
wget https://dist.apache.org/repos/dist/dev/incubator/doris/0.15/0.15.0-rc04/apache-doris-0.15.0-incubating-src.tar.gz

// 或者
https://www.apache.org/dyn/closer.cgi/incubator/doris/$VERSION-incubating/apache-doris-$VERSION-incubating-src.tar.gz

// 或者

git clone https://github.com/apache/incubator-doris.git
```

### 编译 Doris
如果你是第一次使用 `build-env-for-0.15.0` 或之后的版本，第一次编译的时候要使用如下命令。这是因为 `build-env-for-0.15.0` 版本镜像升级了 thrift(0.9 -> 0.13)，需要通过 `--clean` 命令强制使用新版本的 thrift 生成代码文件，否则会出现不兼容的代码。
```bash
sh build.sh --clean --be --fe --ui
```

## 安装 Doris

### 前置准备
```bash
sudo yum install -y java-11-openjdk gcc

cat <<\EOF>> ~/.bashrc
export JAVA_HOME="$(cd "$(dirname $(readlink -f $(which java)))/../../jre" && pwd)"
EOF

source ~/.bashrc

echo $JAVA_HOME
```
限制句柄数：
```bash
vi /etc/security/limits.conf

* soft nofile 65536
* hard nofile 65536
```
检测及关闭系统 swap 分区：
```bash
# 以下会立即生效, 无需重启
echo "vm.swappiness = 0">> /etc/sysctl.conf
swapoff -a && swapon -a
sysctl -p
```

### 配置文件
**IP 绑定（重要）**<br />多网卡时配置参数来让 Doris 根据参数来识别对应的网卡，如果网卡 IP 不正确，无法建立集群。**前后端**都需要修改。
```bash
priority_networks = 10.65.xxx.xxx/24
```
**前端配置**

- 元数据路径，默认 `${DORIS_HOME}/doris-meta`，建议指定外部 SSD
- 修改 JVM 最大堆内存，如果需要的话
```properties
# 路径根据实际自行调节
meta_dir = /data/doris-meta

# 堆内存限制根据实际自行调节
JAVA_OPTS='...'
JAVA_OPTS_FOR_JDK_9='...'
```
**后端配置**

- 数据文件路径

默认 `${DORIS_HOME}/storage`，建议指定外部 SSD<br />可以指定多路径用 `;` 分割，`.SSD` 和 `.HDD` 区分刺配类型，`,10` 表示限制最大容量 GB<br />例如：<br />`storage_root_path=/home/disk1/doris.HDD,50;/home/disk2/doris.SSD,10;/home/disk2/doris`

- 表名大小写敏感性设置

默认为表名大小写敏感, 如有表名大小写不敏感的需求需在集群初始化时进行设置.<br />表名大小写敏感性在集群初始化完成后**不可再修改**。
```properties
# 路径根据实际自行调节
storage_root_path = /data/storage

# 开启不区分大小写
lower_case_table_names = 1
```

### 安装 Doris 前端
拷贝代码到前端节点上，修改配置文件：
```bash
vim conf/fe.conf
```
在**第一个**前端上启动：
```bash
bin/start_fe.sh --daemon
```
启动后用 MySQL 客户端连接该节点，并在客户端内执行以下语句，其中 MySQL 连接的端口是 `grep edit_log_port fe/conf/fe.conf`的端口。
```bash
mysql -h 10.65.62.118 -P 9030 -uroot
```
```bash
# Alive 字段应为 true
SHOW PROC '/frontends';
```
如果有报错，检查进程并查看日志：
```bash
# 检查进程
ps -ef | grep PaloFe

# 查看日志
tail -f log/fe.log
tail -f log/fe.out
```

### 安装 Doris 后端
拷贝代码到后端节点上，修改配置文件：
```bash
vim conf/be.conf
```
用 MySQL 客户端连接前端，并在客户端内执行以下语句，其中

- MySQL 连接的端口是 `grep edit_log_port fe/conf/fe.conf`的端口
- MySQL 执行语句端口是 `grep heartbeat_service_port be/conf/be.conf`的端口
```bash
mysql -h 10.65.62.118 -P 9030 -uroot
```
```bash
ALTER SYSTEM ADD BACKEND "10.65.62.118:9050";
```
然后在后端节点上启动：
```bash
bin/start_be.sh --daemon
```
回到 MySQL 客户端中查看进程：
```bash
# Alive 字段应为 true
SHOW PROC '/backends';
```
如果有报错，检查进程并查看日志：
```bash
# 检查进程
ps -ef | grep palo_be

# 查看日志
tail -f log/be.out
tail -f log/be.INFO
```

### 前端扩容
FE 分为 **Leader**，**Follower** 和 **Observer** 三种角色。默认一个集群，只能有一个 Leader，可以有多个 Follower 和 Observer。其中 Leader 和 Follower 组成一个 Paxos 选择组，如果 Leader 宕机，则剩下的 Follower 会自动选出新的 Leader，保证写入高可用。Observer 同步 Leader 的数据，但是不参加选举。如果只部署一个 FE，则 FE 默认就是 Leader。第一个启动的 FE 自动成为 Leader。在此基础上，可以添加若干 Follower 和 Observer。

- 一个 Leader 和一个 Observer 实现读高可用
- 一个 Leader 和两个 Follower 实现读写高可用
- Follower（包括 Leader）必须是奇数个（官方建议 3 个即可），Observer 可以任意地堆

在主 FE 的 MySQL 客户端中执行以下语句，其中 Host 地址为**新加入 FE 的地址**，端口为各自的 `grep edit_log_port fe/conf/fe.conf`对应的端口。
```bash
ALTER SYSTEM ADD FOLLOWER "10.65.62.118:9010";
ALTER SYSTEM ADD FOLLOWER "10.65.62.119:9010";
ALTER SYSTEM ADD FOLLOWER "10.65.62.120:9010";
ALTER SYSTEM ADD OBSERVER "10.65.62.121:9010";
```
首次启动 Follower 和 Observer 时需要使用以下命令，之后就是用正常启动的命令即可。其中 Host 地址是当**前主 FE 的地址**，端口为 Leader 的 `grep edit_log_port fe/conf/fe.conf`对应的端口。
```bash
bin/start_fe.sh --helper "10.65.62.118:9010" --daemon
```
回到 MySQL 客户端中查看进程：
```bash
SHOW PROC '/frontends';
```

### 后端扩容
在主 FE 的 MySQL 客户端中执行以下语句，其中 MySQL 执行语句端口是 `grep heartbeat_service_port be/conf/be.conf`的端口：
```sql
ALTER SYSTEM ADD BACKEND "10.65.62.119:9050";
ALTER SYSTEM ADD BACKEND "10.65.62.120:9050";
ALTER SYSTEM ADD BACKEND "10.65.62.121:9050";
```
正常启动后端：
```bash
bin/start_be.sh --daemon
```
回到 MySQL 客户端中查看进程：
```bash
SHOW PROC '/backends';
```

### 前端缩容
FE 缩容必须始终保证剩余的 Follower (包括 Leader) 为**奇数个**。<br />在主 FE 的 MySQL 客户端中执行以下语句：
```sql
ALTER SYSTEM DROP FOLLOWER "fe_host:edit_log_port";
ALTER SYSTEM DROP OBSERVER "fe_host:edit_log_port";
```
在对应节点上停止前端：
```bash
bin/stop_fe.sh
```

### 后端缩容
在主 FE 的 MySQL 客户端中执行以下语句：
```sql
ALTER SYSTEM DECOMMISSION BACKEND "be_host:be_heartbeat_service_port";
```
该命令会将被删除节点上的数据逐步迁移到其他节点上后，再自动删除节点。

- 在以上过程中节点的状态始终为 **true**
- 该命令不一定执行成功。例如其他节点上的空间不足，或剩余节点数不满足要求。
```bash
bin/stop_be.sh --daemon
```

## 使用手册
root 默认没有密码，首次使用请手动修改强密码。
```bash
mysql -h 10.65.xxx.xxx -P 9030 -uroot
```
```sql
SET PASSWORD FOR 'root' = PASSWORD('your_password');
```
创建测试数据库：
```sql
CREATE DATABASE example_db;

USE example_db;

CREATE TABLE table1
(
    siteid INT DEFAULT '10',
    citycode SMALLINT,
    username VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(siteid, citycode, username)
DISTRIBUTED BY HASH(siteid) BUCKETS 10
PROPERTIES("replication_num" = "1");

CREATE TABLE table2
(
    event_day DATE,
    siteid INT DEFAULT '10',
    citycode SMALLINT,
    username VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, siteid, citycode, username)
PARTITION BY RANGE(event_day)
(
    PARTITION p201706 VALUES LESS THAN ('2017-07-01'),
    PARTITION p201707 VALUES LESS THAN ('2017-08-01'),
    PARTITION p201708 VALUES LESS THAN ('2017-09-01')
)
DISTRIBUTED BY HASH(siteid) BUCKETS 10
PROPERTIES("replication_num" = "1");
```
```bash
curl --location-trusted -u root \
     -H "label:table1_20170707" \
     -H "column_separator:," \
     -T table1_data \
     http://127.0.0.1:8030/api/example_db/table1/_stream_load

curl --location-trusted -u root \
     -H "label:table2_20170707" \
     -H "column_separator:|" \
     -T table2_data \
     http://127.0.0.1:8030/api/example_db/table2/_stream_load
```
数据文件如下：
```
# table1_20170707 无视本行
1,1,jim,2
2,1,grace,2
3,2,tom,2
4,3,bush,3
5,3,helen,3

# table2_20170707 无视本行
2017-07-03|1|1|jim|2
2017-07-05|2|1|grace|2
2017-07-12|3|2|tom|2
2017-07-15|4|3|bush|3
2017-07-12|5|3|helen|3
```
