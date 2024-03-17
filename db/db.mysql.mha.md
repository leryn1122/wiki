---
id: db.mysql.mha
tags:
- db
- mysql
title: MySQL MHA

---


# MySQL MHA 安装
- [https://www.cnblogs.com/linux-186/p/15245747.html](https://www.cnblogs.com/linux-186/p/15245747.html)
- [https://github.com/yoshinorim/mha4mysql-manager/wiki/](https://github.com/yoshinorim/mha4mysql-manager/wiki/)

这是我司 DBA 的发现目前最好的解决方案

优点:

1. 可以基于 gtid 的复制模式;
2. mha 在进行故障转移时更不易产生数据丢失, 因为主库一旦发生故障了, 如果主库还能 ping 通, MHA 会在主库的/tmp(由参数 remote_workdir 设置)保存主库最后的 binlog, 同时会将该 binlog 传到 manager 节点的 `/etc/masterha/app1` (由参数 manager_workdir 设置)中进行保存. mha 配置 mysql 的半同步复制更能降低数据丢失的概率;
3. 同一个监控节点可以监控多个集群;

缺点:

1. mha 只对主数据库进行监控, 也只有主服务器有 vip;
2. 基于 ssh 免认证配置, 存在一定的安全隐患;
3. 没有提供从服务器的负载均衡;
4. 配置过程比较复杂;
5. 无法避免, 如果监控节点和主库网络出现波动;


## 环境准备
原有的一主二从的 MySQL 集群:
node1 10.xxx.xxx.70 主库 提供写服务
node2 10.xxx.xxx.71 从库 提供读服务
node3 10.xxx.xxx.72 从库 提供读服务
新增一个管理节点:
manager 10.xxx.xxx.73 管理 管理集群
vip 10.xxx.xxx.74 虚拟 IP
```bash
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
```


## 安装 MHA


### SSH 免密登录
配置 ssh 免密登录:
```bash
# 在所有节点把下面的命令都执行一遍
ssh-keygen
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.xxx.xxx.70
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.xxx.xxx.71
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.xxx.xxx.72
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.xxx.xxx.73
```


### 数据节点
```bash
yum install -y perl-DBD-MySQL
rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm
```


### 管理节点
```bash
yum install -y perl-DBD-MySQL
yum install -y perl-Config-Tiny
yum install -y perl-Log-Dispatch
yum install -y perl-Parallel-ForkManager
rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm
rpm -ivh mha4mysql-manager-0.56-0.el6.noarch.rpm
```


## 配置
MySQL**主机**上创建监控账号
```
create user monitor@'%' identified by '123456';
grant all privileges on *.* to 'monitor'@'%';
```
**从机**上设置如下: 因为在默认情况下, 从服务器上的中继日志会在 SQL 线程执行完后被自动删除. 但是在 MHA 环境中, 这些中继日志在恢复其它从服务器时可能会被用到, 因此需要禁用中继日志的自动清除. 改为定期手动清除 SQL 线程应用完的中继日志.
```
set global read_only = 1;
set global relay_log_purge = 0;
```
每个**从机**上: 所有数据库节点建立两个软连接
```bash
ln -s /usr/local/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
```
**管理节点**上: 创建三个 perl 脚本

1. [master_ip_failover](https://www.yuque.com/attachments/yuque/0/2022//26002940/1643083451270-c294db80-57b4-4fa4-b978-f64d1308591b.): 修改 34 行的 ip 地址`my $vip = 'xxx.xxx.xxx.xxx';`
2. [master_ip_online_change](https://www.yuque.com/attachments/yuque/0/2022//26002940/1643083451345-52ecca95-8725-4a3e-b658-dd4427b5e685.): 修改 34 行的 ip 地址`my $vip = 'xxx.xxx.xxx.xxx';`
3. [send_master_failover_mail](https://www.yuque.com/attachments/yuque/0/2022//26002940/1643083451415-4a95aec1-7f97-4009-a911-5e6ae2705798.): 修改 29-33 行的邮箱
```bash
ln -s ~/master_ip_failover        /usr/local/bin/master_ip_failover
ln -s ~/master_ip_online_change   /usr/local/bin/master_ip_online_change
ln -s ~/send_master_failover_mail /usr/local/bin/send_master_failover_mail
chmod u+x master_ip_failover master_ip_online_change send_master_failover_mail
```
在**管理节点**上, 创建集群配置
```bash
mkdir -p /etc/masterha/app1
vi /etc/masterha/app1/app1.cnf
```
无注释详见附件[app1.cnf](https://www.yuque.com/attachments/yuque/0/2022/cnf/26002940/1643083451486-1501d722-0e9e-4c94-b33d-5eb077c1f552.cnf), 根据提示修改其中的 ip 地址, 分清主从
```properties
[server default]
manager_log=/etc/masterha/app1/manager.log      //设置manager的日志
manager_workdir=/etc/masterha/app1              //设置manager的工作目录
master_binlog_dir=/opt/mydata/log/binlog        //设置master默认保存binlog的位置, 以便MHA可以找到master的日志
master_ip_failover_script=/usr/local/bin/master_ip_failover    //设置自动failover时候的切换脚本
master_ip_online_change_script=/usr/local/bin/master_ip_online_change   //设置手动切换时候的切换脚本
user=monitor                                    //设置监控用户
password=123456                                 //设置监控用户的密码
ping_interval=3                                 //设置监控主库, 发送ping包的时间间隔, 默认是3秒, 尝试三次没有回应的时候进行自动failover
remote_workdir=/tmp                             //设置master在发生切换时, 提取binlog的内容所保存的位置, 该目录是在master上
repl_user=repl                                  //设置复制环境中的复制用户名
repl_password=123456                            //设置复制用户的密码
report_script=/usr/local/bin/send_master_failover_mail        //设置发生切换后发送的报警的脚本
secondary_check_script=/usr/bin/masterha_secondary_check -s xxx.xxx.xxx.xxx -s xxx.xxx.xxx.xxx --user=root --master_host=xxx.xxx.xxx.xxx --master_ip=xxx.xxx.xxx.xxx--master_port=3306                              //一旦MHA到master的监控之间出现问题, MHA Manager将会判断其它两个slave是否能建立到master连接, 通过多条网络路由检测master的可用性
shutdown_script=""                              //设置故障发生后关闭故障主机脚本(该脚本的主要作用是关闭主机防止发生脑裂)
ssh_user=root                                   //设置ssh的登录用户名
[server1]
hostname=xxx.xxx.xxx.xxx
port=3306
[server2]
hostname=xxx.xxx.xxx.xxx
port=3306
candidate_master=1                              //设置为候选master, 如果设置该参数以后, 发生主从切换以后将会将此从库提升为主库, 即使这个主库不是集群中最新的slave
check_repl_delay=0                              //默认情况下如果一个slave落后master 100M的relay logs的话, MHA将不会选择该slave作为一个新的master, 因为对于这个slave的恢复需要花费很长时间, 通过设置check_repl_delay=0, MHA触发切换在选择一个新的master的时候将会忽略复制延时, 这个参数对于设置了candidate_master=1的主机非常有用, 因为它保证了这个候选主在切换过程中一定是最新的master
[server3]
hostname=xxx.xxx.xxx.xxx
port=3306
no_master=1                                     //设置这个节点永远不会选为master
```
在 manager 节点上执行下列检查
```bash
# 检查SSH的配置
masterha_check_ssh --conf=/etc/masterha/app1/app1.cnf
# 查看整个集群的状态(从库需启动slave进程)
masterha_check_repl --conf=/etc/masterha/app1/app1.cnf
# 检查MHA Manager的状态
masterha_check_status --conf=/etc/masterha/app1/app1.cnf
```


## 启动
手动开启一个 VIP 网卡
```bash
ifconfig eth0:2 10.xxx.xxx.74 netmask 255.255.255.0 up
```
开启 MHA Manager 监控
```bash
nohup masterha_manager --conf=/etc/masterha/app1/app1.cnf --remove_dead_master_conf --ignore_last_failover >> /etc/masterha/app1/manager.log 2>&1 &
```
remove_dead_master_conf: 该参数代表当发生主从切换后, 老的主库的 IP 将会从配置文件中移除
ignore_last_failover: 在默认情况下, MHA 发生切换后将会在/etc/masterha/app1 下产生 app1.failover.complete 文件, 下次再次切换的时候如果发现该目录下存在该文件且两次切换的时间间隔不足 8 小时的话, 将不允许触发切换. 除非在第一次切换后手动 rm -rf /etc/masterha/app1/app1.failover.complete. 该参数代表忽略上次 MHA 触发切换产生的文件
vip 搭建完成之后并没有 vip, 只有第一次切换之后才会有, 所以所以刚刚配置完 mha 的时候, 如果想用 vip, 需要在主库手工创建一个 vip


### 测试切换
手动下线当前 MySQL 主机:
```bash
service mysql stop
```
查看管理节点上的日志
```bash
tail -f /etc/masterha/app1/manager.log
```
看到日志上发现原 master 宕机, 并且设置将`10.xxx.xxx.71`设置为了新的 master. 登录 slave1 数据库发现确实不再是 slave, 且 slave2 变成了 slave1 节点的 slave
```
master -> down
slave1 -> new master
slave2 -> slave of new master
```
现在重启 master, 将 slave1 上的数据同步给 master, 并且重新建立新的主从关系, 验证数据完整性以及主从同步.
```
master -> slave of new master
slave1 -> new master
slave2 -> slave of new master
```
重新修改配置文件, 并拉起 mha 服务.


### 手工在线切换
```bash
# 关闭监控
masterha_stop --conf=/etc/masterha/app1/app1.cnf
# 会出现两次提示, 需要填yes/no, 填yes即可
masterha_master_switch --conf=/etc/masterha/app1/app1.cnf --master_state=alive --new_master_host=10.xxx.xxx.70 --new_master_port=3306 --orig_master_is_new_slave --running_updates_limit=10000
```
`orig_master_is_new_slave`是将原 master 切换为新主的 slave, 默认情况下, 是不添加的.
`running_updates_limit`默认为 1s, 即如果主从延迟时间(Seconds_Behind_Master), 或 master show processlist 中 dml 操作大于 1s, 则不会执行切换


### 手工 failover
还有种特殊情况是 mha 监控没有开, 但是主库挂掉了该怎么手工 failover
```bash
masterha_check_status --conf=/etc/masterha/app1/app1.cnf
```
在 manager 节点**尝试**进行 failover 切换
```bash
# 提示报错, 主库并没有挂掉, 所以不能failover
masterha_master_switch --conf=/etc/masterha/app1/app1.cnf --master_state=dead --dead_master_host=10.40.16.70 --dead_master_port=3306 --new_master_host=10.xxx.xxx.71 --new_master_port=3306 --ignore_last_failover
```
在 manager 节点再进行 failover 切换
```bash
# 停止MySQL
service mysql stop
# 有两次提示, 都输入yes
masterha_master_switch --conf=/etc/masterha/app1/app1.cnf --master_state=dead --dead_master_host=10.40.16.70 --dead_master_port=3306 --new_master_host=10.40.16.71 --new_master_port=3306 --ignore_last_failover
tail -f /etc/masterha/app1/manager.log
```
这样就完成了手工 failover
