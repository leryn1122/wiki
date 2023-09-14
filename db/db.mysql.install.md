
# MySQL
![](https://img.shields.io/static/v1?label=MySQL&message=2.7.0&color=blue&style=plastic&logo=MySQL&logoColor=white?longCache=true#id=EjlTS&originHeight=18&originWidth=105&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />Reference:

- [MySQL - 官网](https://www.mysql.com/)
- Mysql5.7 - 一键安装脚本 - cnblogs - 作者: 小豹子加油
- Mysql8.0 - 一键安装脚本 - cnblogs - 作者: 小豹子加油

## 包管理器

这很简单..
```bash
sudo apt update
sudo apt install -y mysql-server-8.0 mysql-client-8.0
```

## Docker

使用 dockerhub 上的官方镜像.

```bash
docker run \
  --detach=true \
  --env=MYSQL_USER=mysql \
  --env=MYSQL_ROOT_PASSWORD=root \
  --env=TZ=Asia/Shanghai \
  --publish=3306:3306 \
  --restart=always \
  --volume=/data/mysql:/var/lib/mysql \
  --volume=/data/mysql-files:/var/lib/mysql-files \
  --volume=/conf/mysql:/etc/mysql \
  --name=mysql \
  --hostname=mysql \
  mysql:8.0.26
```

## 二进制安装

### 前置准备
部署准备, 安装前需要准备如下材料:

- MySQL 安装包文件`mysql-*-linux-glibc2.12-x86_64.tar.xz`;
- MySQL 配置文件`my.cnf`, 下面已经准备好了 DBA 调优过的`my.cnf`配置文件;
- MySQL 用户及相关组.
```bash
# 版本自行调整
wget http://dev.mysql.com/get/mysql-8.0.23-linux-glibc2.12-x86_64.tar.xz
wget https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-8.0/mysql-8.0.23-linux-glibc2.12-x86_64.tar.xz
wget https://mirrors.bfsu.edu.cn/mysql/downloads/MySQL-8.0/mysql-8.0.23-linux-glibc2.12-x86_64.tar.xz
```
yum 下载依赖:
```bash
# 部分机器可能没有这个依赖
sudo yum install -y libaio
# 查看是否安装成功
rpm -qa | grep libaio
```
创建 MySQL 用户和相关组:
```bash
sudo groupadd mysql -g 1100
sudo useradd -u 1100 -r -g mysql -s /bin/false mysql
sudo usermod -a -G mysql leryn
```
一些检查工作:

- 检查是否已经卸载了老版本 MySQL, 如果有就卸载.
```bash
sudo rpm -qa | grep -i mysql   | grep -v libs | xargs rpm -ev --nodeps
sudo rpm -qa | grep -i mariadb | grep -v libs | xargs rpm -ev --nodeps
```

### 安装步骤
先做一些 MySQL 在 Linux 系统层面的参数配置.<br />配置 MySQL 资源限制配置文件:
```bash
cat <<\EOF | sudo tee -a /etc/security/limits.conf
mysql    soft    nproc    16384
mysql    hard    nproc    16384
mysql    soft    nofile    65536
mysql    hard    nofile    65536
mysql    soft    stack    1024000
mysql    hard    stack    1024000
EOF
```
配置 Linux 内核配置文件:
```bash
sudo sysctl -w vm.swappiness=5
```
**正式安装**<br />配置 MySQL 环境变量, 并使其生效:
```bash
#!/usr/bin/env bash
# Set MySQL environment.
export MYSQL_HOME=/opt/module/mysql-8.0.23
export PATH=${PATH}:${MYSQL_HOME}/bin
```
创建软链接:
```bash
sudo ln -s ${MYSQL_HOME} /usr/local/mysql
```
解压安装包到指定路径:
```bash
tar -xf mysql-8.0.23-linux-glibc2.12-x86_64.tar.xz
mv mysql-8.0.23-linux-glibc2.12-x86_64 ${MYSQL_HOME}
cd ${MYSQL_HOME}
```
创建数据和日志文件目录(这里修改了后续的`my.cnf`文件中的路径也需要修改):
```bash
mkdir -p /opt/data/mysql/data
mkdir -p /opt/data/mysql/tmp
mkdir -p /opt/data/mysql/log/binlog
mkdir -p /opt/data/mysql/log/relaylog
sudo chown -R mysql:mysql /opt/data/mysql
```
将 MySQL 启动文件移动到系统目录下:
```bash
sudo cp -ar support-files/mysql.server /etc/init.d/mysql
```
修改配置文件`/etc/my.cnf`, 通常 Linux 发行版都会默认自带这个文件, 用如下同事调优的配置参数替换即可:
```bash
sudo cp -ar /etc/my.cnf /etc/my.cnf.bak
sudo vim /etc/my.cnf
```
```properties
[client]
port = 3306
socket = /opt/data/mysql/mysql.sock
default_character_set = utf8mb4
[mysql]
prompt="(\\u@\\h)[\\d]> "
default_character_set = utf8mb4
[mysqldump]
default_character_set = utf8mb4
[mysqld]
# basic settings #
server_id = 128
user = mysql
port = 3306
basedir = /opt/module/mysql-8.0.22
datadir = /opt/data/mysql/data
tmpdir = /opt/data/mysql/tmp
pid_file = /opt/data/mysql/mysql.pid
socket = /opt/data/mysql/mysql.sock
character_set_server = utf8mb4
collation_server = utf8mb4_unicode_ci
autocommit = 0
transaction_isolation = READ-COMMITTED
explicit_defaults_for_timestamp = 1
max_allowed_packet = 1024M
lower_case_table_names = 1
secure_file_priv = ''
open_files_limit = 65535
skip-ssl
default_time_zone='+8:00'
# connection #
skip_name_resolve = 1
max_connections = 1000
max_user_connections = 1000
max_connect_errors = 1000000
thread_cache_size = 512
default_authentication_plugin = mysql_native_password
# memory && myisam #
max_heap_table_size = 128M
tmp_table_size = 128M
join_buffer_size = 16M
key_buffer_size = 64M
bulk_insert_buffer_size = 16M
myisam_sort_buffer_size = 64M
myisam_max_sort_file_size = 6G
myisam_recover_options = DEFAULT
# log settings #
log_error = /opt/data/mysql/log/error.log
log_timestamps = SYSTEM
slow_query_log_file = /opt/data/mysql/log/slowquery.log
slow_query_log = 1
long_query_time = 10
log_queries_not_using_indexes = 1
log_throttle_queries_not_using_indexes = 10
min_examined_row_limit = 100
log_slow_admin_statements = 1
log_bin = /opt/data/mysql/log/binlog/binlog
binlog_format = row
binlog_expire_logs_seconds = 604800
binlog_rows_query_log_events = 1
binlog_row_image = minimal
binlog_cache_size = 8M
max_binlog_cache_size = 2G
max_binlog_size = 1G
log_bin_trust_function_creators = 1
general_log = 0
general_log_file= /opt/data/mysql/log/general.log
# innodb settings #
innodb_data_file_path = ibdata1:1024M:autoextend
innodb_buffer_pool_size = 1G
innodb_lock_wait_timeout = 10
innodb_io_capacity = 4000
innodb_io_capacity_max = 8000
innodb_flush_method = O_DIRECT
innodb_flush_neighbors = 0
innodb_max_undo_log_size = 2G
innodb_log_file_size = 1G
innodb_log_files_in_group = 2
innodb_log_buffer_size = 32M
innodb_thread_concurrency = 16
innodb_print_all_deadlocks = 1
innodb_sort_buffer_size = 16M
innodb_write_io_threads = 4
innodb_read_io_threads = 8
innodb_rollback_on_timeout = 1
innodb_file_per_table = 1
innodb_open_files = 65535
innodb_stats_persistent_sample_pages = 64
innodb_autoinc_lock_mode = 2
# slave #
relay_log = /opt/data/mysql/log/relaylog/relaylog
log_slave_updates = 1
relay_log_purge = 1
relay_log_space_limit = 30G
relay_log_recovery = 1
relay_log_info_repository = TABLE
master_info_repository = TABLE
```
有两个是特殊的配置项:
```properties
# 这个是MySQL的运行内存, 通常在可分配个MySQL内存的60%, 根据机器情况自行调整
innodb_buffer_pool_size
# 单个undo表空间最大值
innodb_max_undo_log_size
# 单个redo日志大小
innodb_log_file_size
# MySQL 8.0改动了账户验证方式, 修改这个参数可以兼容MySQL 8.0
# 使新帐户默认使用本机身份验证, 但只对新建用户生效
# 即使修改原有用户的认证方式, 密码也无法映射为这种验证方式的加密的密码
# 原用户的认证方式仍然保持不变
default_aunthenication_plugin = mysql_native_password
```
初始化 MySQL:
```bash
# 初始化
mysqld --initialize --user=mysql && \
nohup mysqld_safe --user=mysql &
# 检查MySQL进程是否启动
ps -ef | grep mysql | grep -v grep
```
修改 MySQL`root`角色的初始密码:
```bash
# 查看初始密码
sudo grep "temporary password" /opt/data/mysql/log/error.log
=*syBwijr8AT
# root密码重制为root
_passwd=$(grep "temporary password" /opt/data/mysql/log/error.log | awk -F " " '{print $0}') && \
mysqladmin -uroot -p"${_passwd}" password 'root'
# 或者登录MySQL
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
```
安装完成后已经自动启动了 MySQL 服务. 首次启动不能使用`service mysql restart`的方式来停止服务(这疑似一个 bug), 只能进入 MySQL 使用`shutdown`来停止. 此时 MySQL 其实已经安装成功. 如果要修改配置, 需要重新启动 MySQL.
```bash
mysql -uroot -p
# 进入MySQL命令行界面
mysql> shutdown;
mysql> exit;
```

### Boot
启动或停止服务:
```bash
# 设置开机自启动
chkconfig --add mysql
service mysql start
service mysql stop
```
