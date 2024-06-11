---
id: db.tidb
tags:
- db
- tidb
title: "TiDB \u5B89\u88C5\u624B\u518C"

---


# TiDB 安装手册
参考文档：

- [TiDB 产品文档](https://docs.pingcap.com/zh/tidb/stable)
- [TiDB 软件和硬件环境建议配置](https://docs.pingcap.com/zh/tidb/v4.0/hardware-and-software-requirements)
- [最小拓扑架构](https://docs.pingcap.com/zh/tidb/v4.0/minimal-deployment-topology)
- [TiDB 环境与系统配置检查](https://docs.pingcap.com/zh/tidb/v4.0/check-before-deployment)
- [使用 TiUP 部署 TiDB 集群](https://docs.pingcap.com/zh/tidb/v4.0/production-deployment-using-tiup)



## 前置步骤

**_这一块 tuning 有关的内容开发环境安装时都没有做, 不做不会导致安装失败, 但根据文档可能会影响性能._**
参考文档:

- [检查和配置操作系统优化参数 - TiDB 官方文档](https://docs.pingcap.com/zh/tidb/v4.0/check-before-deployment#%E6%A3%80%E6%9F%A5%E5%92%8C%E9%85%8D%E7%BD%AE%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96%E5%8F%82%E6%95%B0)

1. 检测及关闭系统 swap 分区:

```bash
# 以下会立即生效, 无需重启
echo "vm.swappiness = 0">> /etc/sysctl.conf
swapoff -a && swapon -a
sysctl -p
```

2. 检测及安装 NTP 服务, 参考模板机安装手册, 略.
3. 检测及关闭透明大页:

```bash
# 查看透明大页状态
cat /sys/kernel/mm/transparent_hugepage/enabled

# 可能会显示以下
# [always] madvise never

vim /etc/default/grub
# 在 GRUB_CMDLINE_LINUX 这一行的引号内加上  transparent_hugepage=never
# GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet transparent_hugepage=never"

grub2-mkconfig -o /boot/grub2/grub.cfg
```

4. 配置 SSH 免密互信

原官方手册上会推荐使用普通用户, 再 `su - root` 或 `sudo`. 我们选择直接使用 root 用户简化步骤, 不会对应用有任何影响.这步建议使用命令行工具同时对所有 Session 窗口操作, 使机器之间两两互信.

```bash
ssh-keygen -t rsa

# 依次替换成所有节点的 IP
ssh-copy-id -i ~/.ssh/id_rsa.pub 10.0.1.1
```

5. 安装 numactl 工具

> 在生产环境中，因为硬件机器配置往往高于需求，为了更合理规划资源，会考虑单机多实例部署 TiDB 或者 TiKV。
NUMA 绑核工具的使用，主要为了防止 CPU 资源的争抢，引发性能衰退。NUMA 绑核是用来隔离 CPU 资源的一种方法，适合高配置物理机环境部署多实例使用。


```bash
yum -y install numactl
```



## 安装步骤

完成按步骤之后,

1. 先执行以下步骤安装 TiUP:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh

source .bash_profile

# 确认 TiUP 工具是否安装
which tiup

tiup cluster

# 预期显示 Update successfully! 字样
tiup update --self && tiup update cluster

# 查看 TiUP cluster 组件版本
tiup --binary cluster
```

2. 选一台机器作为中控机, 之后到操作只需要在中控机上操作即可. 在其上编写一个拓扑结构的 YAML 文件`topology.yaml`. 参考这个配置 [标准环境拓扑架构 YAML](https://github.com/pingcap/docs-cn/blob/release-4.0/config-templates/complex-mini.yaml). 只需要写出对应组件安装的 IP, 端口, 文件路径等等, 不填写则使用注释中的默认值. 如果同一节点上需要安装多个相同组件时, 需要区分端口和文件路径.

```bash
vim topology.yaml
```

```yaml
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
global:
  user: 'tidb'
  ssh_port: 22
  deploy_dir: '/tidb-deploy'
  data_dir: '/tidb-data'

# # Monitored variables are applied to all the machines.
monitored:
  node_exporter_port: 9100
  blackbox_exporter_port: 9115
  # deploy_dir: "/tidb-deploy/monitored-9100"
  # data_dir: "/tidb-data/monitored-9100"
  # log_dir: "/tidb-deploy/monitored-9100/log"

# # Server configs are used to specify the runtime configuration of TiDB components.
# # All configuration items can be found in TiDB docs:
# # - TiDB: https://pingcap.com/docs/stable/reference/configuration/tidb-server/configuration-file/
# # - TiKV: https://pingcap.com/docs/stable/reference/configuration/tikv-server/configuration-file/
# # - PD: https://pingcap.com/docs/stable/reference/configuration/pd-server/configuration-file/
# # All configuration items use points to represent the hierarchy, e.g:
# #   readpool.storage.use-unified-pool
# #
# # You can overwrite this configuration via the instance-level `config` field.

server_configs:
  tidb:
    log.slow-threshold: 300
    binlog.enable: false
    binlog.ignore-error: false
  tikv:
    # server.grpc-concurrency: 4
    # raftstore.apply-pool-size: 2
    # raftstore.store-pool-size: 2
    # rocksdb.max-sub-compactions: 1
    # storage.block-cache.capacity: "16GB"
    # readpool.unified.max-thread-count: 12
    readpool.storage.use-unified-pool: false
    readpool.coprocessor.use-unified-pool: true
  pd:
    schedule.leader-schedule-limit: 4
    schedule.region-schedule-limit: 2048
    schedule.replica-schedule-limit: 64

pd_servers:
  - host: 10.0.1.4
    # ssh_port: 22
    # name: "pd-1"
    # client_port: 2379
    # peer_port: 2380
    # deploy_dir: "/tidb-deploy/pd-2379"
    # data_dir: "/tidb-data/pd-2379"
    # log_dir: "/tidb-deploy/pd-2379/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.pd` values.
    # config:
    #   schedule.max-merge-region-size: 20
    #   schedule.max-merge-region-keys: 200000
  - host: 10.0.1.5
  - host: 10.0.1.6

tidb_servers:
  - host: 10.0.1.1
    # ssh_port: 22
    # port: 4000
    # status_port: 10080
    # deploy_dir: "/tidb-deploy/tidb-4000"
    # log_dir: "/tidb-deploy/tidb-4000/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.tidb` values.
    # config:
    #   log.slow-query-file: tidb-slow-overwrited.log
  - host: 10.0.1.2
  - host: 10.0.1.3

tikv_servers:
  - host: 10.0.1.7
    # ssh_port: 22
    # port: 20160
    # status_port: 20180
    # deploy_dir: "/tidb-deploy/tikv-20160"
    # data_dir: "/tidb-data/tikv-20160"
    # log_dir: "/tidb-deploy/tikv-20160/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.tikv` values.
    # config:
    #   server.grpc-concurrency: 4
    #   server.labels: { zone: "zone1", dc: "dc1", host: "host1" }
  - host: 10.0.1.8
  - host: 10.0.1.9

monitoring_servers:
  - host: 10.0.1.10
    # ssh_port: 22
    # port: 9090
    # deploy_dir: "/tidb-deploy/prometheus-8249"
    # data_dir: "/tidb-data/prometheus-8249"
    # log_dir: "/tidb-deploy/prometheus-8249/log"

grafana_servers:
  - host: 10.0.1.10
    # port: 3000
    # deploy_dir: /tidb-deploy/grafana-3000

alertmanager_servers:
  - host: 10.0.1.10
    # ssh_port: 22
    # web_port: 9093
    # cluster_port: 9094
    # deploy_dir: "/tidb-deploy/alertmanager-9093"
    # data_dir: "/tidb-data/alertmanager-9093"
    # log_dir: "/tidb-deploy/alertmanager-9093/log"
```

3. 准备好配置文件后

```bash
tiup cluster deploy tidb-test v4.0.16 ./topology.yaml --user root
```

以上部署命令中：

- 通过 TiUP cluster 部署的集群名称为 `tidb-test`
- 部署版本为 `v4.0.16`, 最新版本可以通过执行`tiup list tidb` 来查看 TiUP 支持的版本
- 初始化配置文件为 `topology.yaml`
- `--user root`: 通过 root 用户登录到目标主机完成集群部署, 该用户需要有 ssh 到目标机器的权限, 并且在目标机器有 sudo 权限. 也可以用其他有 ssh 和 sudo 权限的用户完成部署.

预期日志结尾输出会有 `Deployed cluster`tidb-test`successfully` 关键词, 表示部署成功.


4. 启动集群并初始化数据库, 初始化成功后会显示, 初始 root 密码, 密码请**务必**保存, 只会出现一次.

```bash
tiup cluster start tidb-test --init
```

5. 验证集群运行状态:

```bash
tiup cluster display tidb-test
```

6. 验证数据库连接, 本地没有 mysql 客户端, 请使用其他地方的客户端或数据库工具连接:

```bash
mysql -u root -h <YOUR_IP_ADDRESS> -P 4000
```




# 备份数据

```bash
tiup install dumpling

tiup dumpling  -u root -P 3306 -h <YOUR_IP_ADDRESS> -p <YOUR_IP_PASSWORD> --filetype sql -t 8 -o tidb-dump.sql -r 200000 -F 256MiB -B demo

```
