
# MySQL 主从搭建手册
参考文档.

- [https://www.cnblogs.com/ddzj01/p/10678296.html](https://www.cnblogs.com/ddzj01/p/10678296.html)
- [https://www.cnblogs.com/ddzj01/p/14397698.html](https://www.cnblogs.com/ddzj01/p/14397698.html)

## 环境准备

准备至少两台已安装 MySQL 的服务器，且保证两台服务器能够互通。

## 主从搭建

### 修改配置文件
在从库中修改配置文件，修改 MySQL 配置文件。
```bash
# 修改前配置文件前需要停止MySql服务
service mysql stop;
vi /etc/my.cnf
```

- 修改`server_id`：主从的`server_id`不同（主从`server_id`必须不同且从从之间不相同，非主从关系无所谓），通常使用 IP 地址的最后两位拼接作为`server_id`，例如地址为`xxx.xxx.123.45`，那么`server_id = 12345`。**注意: 修改**`**server_id**`**前如果没有停止 MySql 服务，重启 MySql 服务将不能正确找到 PID 文件并停止 MySql。**
- 修改`innodb_buffer_pool_size`：这是 MySQL 的运行内存，一般在可分配给 MySQL 内存的 60%，根据每个机器具体资源调整。
- 检查`default_aunthenication_plugin=mysql_native_password`是否配置正确，若不是则改为该配置，为了身份验证。MySQL 8.0 改动了账户验证方式，修改这个参数可以兼容 MySql 8.0 使新帐户默认使用本机身份验证，但只对新建用户生效。原用户的认证方式仍然保持不变。

使服务器环境变量生效：
```bash
source .bash_profile
```
**从库**修改后`server_id`后, 重启 MySQL 服务：
```bash
service mysql start
```
登录数据库，若成功则验证 MySQL 安装成功：
```bash
mysql -uroot -p
```

### 备份主库 && 导入主库备份文件
查看**主库**的`binlog`
```bash
mysql -uroot -p -e "show master status\G" > master-binlog.txt
```
需要保存输出内容`File`和`Position`两个字段, 后面会使用到。执行命令备份主库导出数据文件（全库备份）：
```bash
mysqldump -uroot -p --flush-logs --all-databases --single-transaction > all.sql
```
查看备份文件的内容：
```bash
less all.sql
```
将**主库**服务器导出的数据文件通过`scp`发送到从库服务器：
```bash
scp all.sql 从库服务器IP地址:~
```
在**从库**服务器导入先前备份的数据库文件：
```bash
mysql -uroot -p < all.sql
```
登录**从库**：
```bash
mysql -uroot -p
```
检查备份数据库是否导入：
```sql
mysql> show databases;
```

### 建立主从关系
在**主库**中创建一个用户专门用于复制数据：
```sql
mysql> create user repl@'%' identified by '密码';
mysql> grant replication slave, replication client on *.* to 'repl'@'%';
```
从库的配置文件需要新增一个属性, 并重启 MySQL 实例：
```properties
# /etc/my.cnf
read_only = 1
```
在**从库**中执行命令绑定主库，`MASTER_LOG_FIE`和`MASTER_LOG_POS`必须和主库备份状态保持一致：
```
# 先前保存的master status的结果
MASTER_LOG_FIE <== File
MASTER_LOG_POS <== Position
```
```sql
# 若在从库修改绑定主库信息必须要先停止slave
mysql> stop slave;
mysql> change master to
         master_host='主库IP地址',
         master_port=3306,
         master_user='repl',
         master_password='密码',
         master_log_file='先前记录的主库MASTER_LOG_FIE',
         master_log_pos=先前记录的主库MASTER_LOG_POS;
mysql> start slave;
```
查看**从库**的状态，检查是否存在`error`和`warning`。
```plsql
mysql> show slave status\G
```
如果有这样的记录，说明主从配置有问题：
```
Last_IO_Error: error connecting to master ...
```
设置从库只读，如果`my.cnf`已经设置了，就不需要重复配置<br />值得注意的是, 在将主库数据导入前不能开启`super_read_only`，否则会无法导入数据，而每次修改`my.cnf`都需要重启数据库，并且重启后要手动`start slave`。
```sql
# 如果my.cnf配置了中配置了read_only = 1, 那么以下结果中的read_only和super_read_only为ON
mysql> show global variables like "%read_only%";
# 或者临时设置全局变量
mysql> set global read_only = 1;
mysql> set global super_read_only = 1;
```
```
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | ON    |
| super_read_only       | ON    |
| transaction_read_only | OFF   |
+-----------------------+-------+
```

## 常见问题
**repl 用户认证**<br />检查用户认真方式是否为`mysql_native_password`：
```sql
mysql> select user, host, plugin from mysql.user;
```
如果是请修改为`mysql_native_password`，并重新修改密码。

**同步出错**<br />参考文档：

- [https://dba.stackexchange.com/questions/10184/mysql-replication-error](https://dba.stackexchange.com/questions/10184/mysql-replication-error)

如果 `start slave` 后发现有以下报错信息：
```
Worker 1 failed executing transaction 'ANONYMOUS' at master log binlog.003091, end_log_pos 333974; Could not execute Write_rows event on table xxxxxxx; Duplicate entry '5910143' for key 'xxxxxxx', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log binlog.003091, end_log_pos 333974
```
或者：
```
Coordinator stopped because there were error(s) in the worker(s). The most recent failure being: Worker 1 failed executing transaction 'ANONYMOUS' at master log binlog.003091, end_log_pos 335667. See error log and/or performance_schema.replication_applier_status_by_worker table for more details about this failure or others, if any.
```
第二种情况下根据提示查询：
```sql
select * from performance_schema.replication_applier_status_by_worker t
 where t.last_applied_transaction = 'ANONYMOUS'
```
那么这个时候说明同步时出现了数据不一致的问题，目前猜测可能的原因是 `mysqldump` 时主库仍然在写入，导致的了不一致（`--single-transaction`为何不生效？？），根据报错信息解决数据记录，然后当前跳过同步记录：例如 `HA_ERR_FOUND_DUPP_KEY` 主键重复报错，查看两边数据一致后可以跳过这条同步记录。
```sql
# mysql_skip.sql
stop slave;

set global SQL_SLAVE_SKIP_COUNTER = 1;

start slave;

select SLEEP(1);
```
```bash
# mysql_skip.sh

#!/bin/bash

dup_status=`mysql -uroot -p******* --execute="show slave status" | grep HA_ERR_FOUND_DUPP_KEY | wc -l`
if [ $dup_status -eq 0 ]; then
  dup_status=`mysql -uroot -p******* --execute="select t.last_error_message from performance_schema.replication_applier_status_by_worker t where t.last_applied_transaction = 'ANONYMOUS'"|grep HA_ERR_FOUND_DUPP_KEY|wc -l`
fi

while [ $dup_status -ne 0 ]; do
  mysql -uroot -p******* < mysql_skip.sql
  dup_status=`mysql -uroot -p******* --execute="show slave status" | grep HA_ERR_FOUND_DUPP_KEY | wc -l`
  if [ $dup_status -eq 0 ]; then
    dup_status=`mysql -uroot -p******* --execute="select t.last_error_message from performance_schema.replication_applier_status_by_worker t where t.last_applied_transaction = 'ANONYMOUS'"|grep HA_ERR_FOUND_DUPP_KEY|wc -l`
  fi
  echo $dup_status
done
```
可以保存这两个脚本，并且运行 `mysql_skip.sh` 即可。
