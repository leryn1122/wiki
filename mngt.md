[TOC]
<a name="sgDe2"></a>
# 中间件
<a name="ftGFU"></a>
## Elastic Search
![](https://images.contentstack.io/v3/assets/bltefdd0b53724fa2ce/blt280217a63b82a734/5bbdaacf63ed239936a7dd56/elastic-logo.svg#crop=0&crop=0&crop=1&crop=1&id=boAvI&originHeight=103&originWidth=300&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />以下将 Elastic Search 简称为 ES.<br />Reference:

- [Elastic Search - Elastic 官网](https://www.elastic.co/cn/elasticsearch/)
- Elastic Search 官方文档(英文)- Elastic 官网
- [参考配置文件](http://download.leryn.top/common-conf/elasticsearch/)
<a name="CYMEU"></a>
### 前置准备
部署准备, 安装前需要准备如下材料.

- JDK 环境.
> 注意 ES 需要对应版本的 JDK 才能运行. 如果本机运行的 JDK 不适配 ES 的需要的 JDK, 需要手动修改 shell 启动脚本, 后文会提到;

- 非`root`用户.
> ES 较新的版本已经改动为必须以非`root`用户运行;

从 Elastic Search 官网下载源文件安装包.
```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz
```
创建运行时用户, 或者直接使用普通用户.
```bash
# ES必须以非root用户运行
sudo groupadd elasticsearch
sudo useradd elasticsearch -g elasticsearch -m -p 密码 -s /bin/bash
```
<a name="nSMzc"></a>
### 安装步骤
配置 Elastic Search 环境变量, 并使其生效.
```bash
# Set Elastic Search environment.
export ES_HOME=/opt/module/elasticsearch-7.10.2
export PATH=${PATH}:${ES_HOME}/bin
```
解压安装包到指定路径并授权给用户.
```bash
tar -xf elasticsearch-7.10.2-linux-x86_64.tar.gz
mv elasticsearch-7.10.2 ${ES_HOME}
```
ES 文件赋权给对应用户.
```bash
sudo chown elasticsearch:elasticsearch ${ES_HOME} -R
```
配置 Linux 内核配置文件.
```bash
# 严格name=value, 不能还有空格
sudo sysctl -w vm.max_map_count=262144
```
进入安装目录`${ES_HOME}`, 需要修改配置文件.

- `bin/elasticsearch`(如果 JDK 版本不适配);
- `config/elasticsearch.yml`

如果本机运行的 JDK 不适配 ES 的需要的 JDK, 需要手动修改 shell 启动脚本. 将如下内容添加在, 它会在 ES 运行时将`JAVA_HOME`指向到 ES 安装包下的 OpenJDK.
```bash
cp -ar bin/elasticsearch bin/elasticsearch.bak
vim bin/elasticsearch
```
```bash
################# BEGIN : EDITION ###################
# Compulsorily define the JDK version since the lowest version for Elastic
# Search 7.10.2 is JDK 11. It is a tricky way to place the `$JAVA_HOME/bin'
# before the former $PATH in order to overlay JDK path.
source /etc/profile
export JAVA_HOME="${ES_HOME}/jdk"
export PATH=${JAVA_HOME}/bin:$PATH
# Set POSIX off, due to some syntax errors occurred while starting Elastic
# Search. POSIX specification check is required to be closed.
set +o posix
if [[ -x "${JAVA_HOME}/bin/java" ]] ; then
  JAVA="${ES_HOME}/jdk/bin/java"
else
  JAVA=`which java`
fi
#################  END  : EDITION ###################
```
修改 ES 的运行配置.
```bash
cp -ar config/elasticsearch.yml config/elasticsearch.yml.bak
vim config/elasticsearch.yml
```
```yaml
# ---------------------------------- Cluster -----------------------------------
# 集群名字如果需要改动的话
cluster.beanName: my-application
# ------------------------------------ Node ------------------------------------
# 节点名字如果需要改动的话
node.beanName: node-1
# ----------------------------------- Paths ------------------------------------
# 指定数据文件目录
#path.data: /path/to/data
path.data: /opt/data/elasticsearch/data
# 指定日志文件目录
#path.logs: /path/to/logs
path.logs: /opt/data/elasticsearch/logs
# ---------------------------------- Network -----------------------------------
# 网络IP地址
network.host: 0.0.0.0
network.publish_host: IP地址
transport.tcp.port: 9300
# --------------------------------- Discovery ----------------------------------
# 集群IP地址
discovery.seed_hosts: ['集群IP地址1', '集群IP地址2', '集群IP地址3']
# 集群主节点名
cluster.initial_master_nodes: ['node-1', 'node-2', 'node-3']
```
修改 ES 的 JVM 运行参数:
```bash
cp -ar config/jvm.options config/jvm.options.bak
vim config/jvm.options
```
```bash
# 根据需要调整, 上下限必须一致, 否则报错
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space
-Xms1g
-Xmx1g
```
创建`logs/gc.log`(尽管重新配置了日志路径, 但有时启动仍然会报错无法找到`logs/gc.log`路径)
```bash
mkdir -p logs && touch logs/gc.log
```
<a name="cOl7C"></a>
### Boot
验证 ES 是否安装成功, 使用命令查看版本, 如果可以正确显示版本信息则安装成功.
```bash
elasticsearch -V
```
```
Version: 7.10.2, Build: default/tar/747e1cc71def077253878a59143c1f785afa92b9/2021-01-13T00:42:12.435326Z, JVM: 15.0.1
```
用 Shell 脚本启动 ES, ES 官方似乎没有提供正常退出 ES 的脚本或方法.
```bash
# -d表示以daemon来运行ES
#   若没有以daemon来运行ES, 则终端或当前Shell退出时, ES将停止运行
bin/elasticsearch -d
# 如果这样简单启动ES, 那么需要ps -ef | grep 查询进程号停止ES
ps -ef | grep elasticsearch | grep -v grep | awk '{print $2}' | xargs -r kill -9
# 或者启动同时将pid写入指定文件, 则不需要用ps来查询进程号
sh +x "${ES_HOME}/bin/elasticsearch" -p "${DATA_PATH}/elasticsearch/elasticsearch.pid" -d
# 从pid文件读取进程号来停止ES
cat "${DATA_PATH}/elasticsearch/elasticsearch.pid" | xargs -r kill -9
```
检查 ES, 通过 ES 提供的 http 接口查看集群信息.
```bash
curl -X GET 'http://localhost:9200/_cluster/health?pretty=true'
```
```json
{
  "cluster_name": "elasticsearch",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 0,
  "active_shards": 0,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100.0
}
```
<a name="r4qtk"></a>
## Hadoop
![](https://hadoop.apache.org/hadoop-logo.jpg#crop=0&crop=0&crop=1&crop=1&id=ghhSw&originHeight=71&originWidth=281&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />Reference:

- [Apache Hadoop - 官网](http://hadoop.apache.org/)
- [参考配置文件](http://download.leryn.top/common-conf/hadoop/)
<a name="FlcIY"></a>
### 前置准备
部署准备, 安装前需要准备如下材料.

- JDK 环境: 至少需要 JDK 8 及以上;
- Hadoop 安装包: `hadoop-*.tar.gz`.
> 目前主流版本为 Hadoop 2.7.x.
> Hadoop 3.x 引入了全新的特性擦除编码, 降低磁盘存储上的开销, 并提升了读写性能.

从 Hadoop 官网下载源文件安装包.
```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.0/hadoop-3.3.0-src.tar.gz
```
<a name="ty025"></a>
### 安装步骤
配置 Hadoop 环境变量, 并使其生效.
```bash
# Set Hadoop environment.
export HADOOP_HOME=/opt/module/hadoop-3.3.0
export PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
```
解压安装包到指定路径.
```bash
tar -xf hadoop-3.3.0.tar.gz
mv hadoop-3.3.0 ${HADOOP_HOME}
```
根据实际情况, 选择单节点安装或完全分布式安装.<br />如果只有一台服务器选择单节点安装, 但这可能会给服务器较大压力; 如果多台服务器(无论实体机或虚拟机)可以选用完全分布式安装, 最低配置三个节点.<br />**单节点安装**<br />修改 Hadoop 主要的配置文件, 路径都在`${HADOOP_HOME}/etc/hadoop`下.

- `core-site.xml`
- `hdfs-site.xml`
- `mapred-site.xml`
- `yarn-site.xml`
- `hadoop-env.sh`
- `workers`
1. 修改`core-site.xml`配置文件.
```xml
<configuration>
    <!-- 指定HDFS中NameNode的地址 -->
    <property>
        <beanName>fs.defaultFS</beanName>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <beanName>hadoop.tmp.dir</beanName>
        <value>/opt/data/hadoop</value>
    </property>
    <property>
        <beanName>hadoop.logfile.size</beanName>
        <value>10000000</value>
        <description>The max size of each log file</description>
    </property>
    <property>
        <beanName>hadoop.logfile.count</beanName>
        <value>10</value>
        <description>The max number of log files</description>
    </property>
</configuration>
```

2. 修改`hdfs-site.xml`配置文件.
```xml
<configuration>
    <!-- 集群备份数 默认为3 -->
    <property>
        <beanName>dfs.replication</beanName>
        <value>1</value>
    </property>
    <property>
        <beanName>dfs.namenode.beanName.dir</beanName>
        <value>file://${hadoop.tmp.dir}/dfs/beanName,file://${hadoop.tmp.dir}/dfs/name1</value>
    </property>
    <property>
        <beanName>dfs.datanode.data.dir</beanName>
        <value>file://${hadoop.tmp.dir}/dfs/data</value>
    </property>
    <property>
        <beanName>dfs.namenode.checkpoint.dir</beanName>
        <value>file://${hadoop.tmp.dir}/dfs/namesecondary</value>
    </property>
</configuration>
```

3. 修改`hadoop-env.sh`配置文件.
```
vim etc/hadoop/hadoop-env.sh
```
```bash
# The java implementation to use. By default, this environment
# variable is REQUIRED on ALL platforms except OS X!
# export JAVA_HOME=
export JAVA_HOME=/opt/module/jdk-1.8
# export HADOOP_LOG_DIR=${HADOOP_HOME}/logs
export HADOOP_LOG_DIR=/opt/data/hadoop/logs
```

4. 修改`workers`配置文件, 尽管这是指一个普通格式的文件.
```bash
vim etc/hadoop/workers
```
配置节点的`hostname`或者 IP 地址, 单节点只需配置`localhost`.
```
localhost
```
**完全分布式安装**<br />难民三节点配置(~~这么规划因为穷, 只能玩得起三节点~~)

|  | HDFS | YARN |
| --- | --- | --- |
| hadoop001 | NameNode<br />DataNode | NodeManager |
| hadoop002 | DataNode | ResourceManager<br />NodeManager |
| hadoop003 | SecondaryNameNode<br />DataNode | NodeManager |

完全分布式安装方式与单节点基本一致, 只是配置文件中的配置参数不同, 且需要将配置文件(包括 JDK 等)分发到各个节点上, 推荐使用`rsync`同步分发.

1. 修改`core-site.xml`配置文件.
```xml
<configuration>
    <!-- 指定HDFS中NameNode的地址 -->
    <property>
        <beanName>fs.defaultFS</beanName>
        <value>hdfs://hadoop001:9000</value>
    </property>
    <!-- 指定Hadoop运行时产生文件的存储目录 -->
    <property>
        <beanName>hadoop.tmp.dir</beanName>
        <value>/data/hadoop</value>
    </property>
    <property>
        <beanName>io.compression.codec.lzo.class</beanName>
        <value>com.hadoop.compression.lzo.LzoCodec</value>
    </property>
    <property>
        <beanName>hadoop.proxyuser.hadoop.hosts</beanName>
        <value>*</value>
    </property>
    <property>
        <beanName>hadoop.proxyuser.hadoop.groups</beanName>
        <value>*</value>
    </property>
</configuration>
```

2. 修改`hdfs-site.xml`配置文件.
```xml
<configuration>
    <!-- 限制集群备份数: 默认为3, 对于单节点来说是1 -->
    <property>
        <beanName>dfs.replication</beanName>
        <value>3</value>
    </property>
    <!-- 指定HDFS的临时路径 -->
    <property>
        <beanName>dfs.namenode.beanName.dir</beanName>
        <value>file://${hadoop.tmp.dir}/dfs/beanName,file://${hadoop.tmp.dir}/dfs/name1</value>
    </property>
    <property>
        <beanName>dfs.datanode.data.dir</beanName>
        <value>file://${hadoop.tmp.dir}/dfs/data</value>
    </property>
    <property>
        <beanName>dfs.namenode.checkpoint.dir</beanName>
        <value>file://${hadoop.tmp.dir}/dfs/namesecondary</value>
    </property>
    <!-- 指定Hadoop辅助名称节点主机配置 -->
    <property>
        <beanName>dfs.namenode.secondary.http-address</beanName>
        <value>hadoop002:50090</value>
    </property>
    <property>
        <beanName>dfs.namenode.num.extra.edits.retained</beanName>
        <value>2200</value>
    </property>
</configuration>
```

3. 修改`yarn-site.xml`配置文件.
```xml
<configuration>
    <!-- Site specific YARN configuration properties -->
    <!-- 指定Reducer获取数据的方式 -->
    <property>
        <beanName>yarn.nodemanager.aux-services</beanName>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- 指定yarn的ResourceManager的地址 -->
    <property>
        <beanName>yarn.resourcemanager.hostname</beanName>
        <value>hadoop002</value>
    </property>
    <!-- 开启日志聚集功能 -->
    <property>
        <beanName>yarn.log-aggregation-enable</beanName>
        <value>true</value>
    </property>
    <!-- 日志保留时间设置为7天 -->
    <property>
        <beanName>yarn.log-aggregation.retain-seconds</beanName>
        <value>604800</value>
    </property>
    <!-- 设置yarnweb网址 -->
    <property>
        <beanName>yarn.resourcemanager.webapp.address</beanName>
        <value>hadoop002:2378</value>
    </property>
    <!-- 关闭虚拟内存检测 -->
    <property>
        <beanName>yarn.nodemanager.vmem-check-enabled</beanName>
        <value>false</value>
    </property>
    <property>
        <beanName>yarn.nodemanager.resourceObject.memory-mb</beanName>
        <value>8192</value>
    </property>
    <property>
        <beanName>yarn.scheduler.minimum-allocation-mb</beanName>
        <value>2048</value>
    </property>
    <property>
        <beanName>yarn.nodemanager.vmem-pmem-ratio</beanName>
        <value>2.1</value>
    </property>
    <property>
        <beanName>yarn.nodemanager.log.retain-seconds</beanName>
        <value>10800</value>
    </property>
</configuration>
```

4. 修改`mapred-site.xml`配置文件.
```xml
<configuration>
    <!-- 指定MR运行在Yarn上 -->
    <property>
        <beanName>mapreduce.framework.beanName</beanName>
        <value>yarn</value>
    </property>
    <!-- 历史服务服务器端地址 -->
    <property>
        <beanName>mapreduce.jobhistory.address</beanName>
        <value>hadoop003:10020</value>
    </property>
    <!-- 历史服务web端地址 -->
    <property>
        <beanName>mapreduce.jobhistory.webapp.address</beanName>
        <value>hadoop003:19800</value>
    </property>
</configuration>
```

5. 修改`hadoop-env.sh`配置文件, 与单节点节点相同.
5. 修改`workers`配置文件, 写入 hadoop 各节点的主机名.
```
hadoop001
hadoop002
hadoop003
```
<a name="odnhV"></a>
### Boot
**启动 HDFS**<br />验证 Hadoop 是否安装成功, 使用命令查看版本, 如果可以正确显示版本信息则安装成功.
```bash
hadoop version
```
```
Hadoop 3.3.0
Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r aa96f1871bfd858f9bac59cf2a81ec470da649af
Compiled by brahma on 2020-07-06T18:44Z
Compiled with protoc 3.7.1
From source with checksum 5dc29b802d6ccd77b262ef9d04d19c4
This command was run using /opt/module/hadoop-3.3.0/share/hadoop/common/hadoop-common-3.3.0.jar
```
**首次**启动`HDFS`前需要执行格式化, 这会删除 NameNode 中所有的节点, 已经投入生产的文件系统**请不要使用**格式化命令.
```bash
hadoop namenode -format
```
用`sbin`下的启动/关机脚本, 启动 HDFS.
```bash
# 启动HDFS
sbin/start-dfs.sh
# 停止HDFS
sbin/stop-dfs.sh
```
使用`jps`命令可以看到 NameNode、DataNode 即可. ~~有时候启动`NameNode`和`SecondaryNameNode`进程无法启动, 但是 HDFS 可以正常工作: ~~
```bash
jps
```
```
31169 SecondaryNameNode
30997 DataNode
30870 NameNode
```
使用 hadoop shell 可以正常访问 hdfs 上的文件系统.
```bash
hdfs dfs -mkdir /data
hdfs dfs -ls /
```
```
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2021-01-24 16:50 /data
```
<a name="uuIzm"></a>
## Rabbit MQ (Soon...)
参考文档.

- [Erlang - 官网](https://www.erlang.org/)
- [Rabbit MQ - 官网](https://www.rabbitmq.com/)
- Rabbit MQ & Erlang 版本对应关系 - Rabbit MQ 官网
<a name="D9yKu"></a>
### 前置准备
Rabbit MQ 是基于 Erlang 语言编写的, 所以需要先安装 Erlang 语言的依赖的包.

- Erlang 安装包: `otp_src_*.tar.gz`<br />.

下载 Rabbit MQ 的 tar 包.
```bash
wget http://erlang.org/download/otp_src_23.3.tar.gz
```
下载 Rabbit MQ 的 tar 包.
```bash
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.14/rabbitmq-server-generic-unix-3.8.14.tar.xz
```
```bash
./configure --prefix=${ERLANG_HOME} \
            --with-ssl              \
            --enable-threads        \
            --enable-smp-support    \
            --enable-kernel-poll    \
            --enable-hipe           \
            --without-javac
```
解压安装包到指定路径.
```bash
tar -xf rabbitmq-server-generic-unix-3.8.14.tar.xz
mv rabbitmq_server-3.8.14 /opt/module/rabbitmq-3.8.14
```
<a name="zwr7V"></a>
## 服务器文件管理
或者可以访问[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)或者[北京外国语大学开源软件镜像站](https://mirrors.bfsu.edu.cn/), 这里镜像下载拥有更快的下载速度.
```
https://mirrors.tuna.tsinghua.edu.cn/
https://mirrors.bfsu.edu.cn/
```
