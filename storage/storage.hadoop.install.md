<a name="r4qtk"></a>
## Hadoop
Reference:

- 
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
6. 修改`workers`配置文件, 写入 hadoop 各节点的主机名.
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
