<a name="748df1c9"></a>
## MySQL生产备份

备份服务器不需要安装mysql数据库, 但需要有mysql的安装包 (需要用到 `mysqldump` 命令).

```bash
# 移除linux发行版内置的mariadb
rpm -qa | grep -i mysql   | grep -v libs | xargs rpm -ev --nodeps &> /dev/null
rpm -qa | grep -i mariadb | grep -v libs | xargs rpm -ev --nodeps &> /dev/null

# 安装mysql必要的依赖包
yum install -y libaio &> /dev/null

# 下载mysql安装包并安装
wget http://dev.mysql.com/get/mysql-8.0.22-linux-glibc2.12-x86_64.tar.xz
tar -xf http://dev.mysql.com/get/mysql-8.0.22-linux-glibc2.12-x86_64.tar.xz -C /usr/local
cd /usr/local
ln -s mysql-*-linux-glibc2.12-x86_64 mysql

# bash_profile中添加一句export语句
vim ~/.bash_profile
# export PATH=$PATH:/usr/local/mysql/bin

# 检查安装
source ~/.bash_profile
which mysql
```

先现在各个数据库建立专门用于备份的用户:

```sql
/* 创建用户，专门用于备份 */
CREATE USER 'datasource_backup'@'%' IDENTIFIED BY '密码';
GRANT SELECT, PROCESS, RELOAD, SUPER, REPLICATION CLIENT, EVENT ON *.* TO 'datasource_backup'@'%';
FLUSH PRIVILEGES;
```

从各数据库主库备份到统一的备份服务器路径下 `/data` , 先 `df -h` 检查 `/data` 是否处于一个空间足够大的逻辑卷上, 否则先参考硬盘扩容文档扩容: 

```bash
# 新建目录
mkdir -p /data
mkdir -p /data/script  # 这个目录用于存放脚本

# 创建创建备份脚本
cd /data/script
touch back.sh 
vim back.sh       # 内容见文末附件

crontab -e        # 添加定时任务

# 添加如下记录，时间自行调整
# 数据库IP 用户 密码 以参数形式依次传给shell脚本
5 3 * * * bash /data/script/back.sh 数据库IP 用户 密码
```

脚本功能：

- 远程读取 mysql 中的所有schema (包括mysql, 但除去三个元数据schema)
- 根据远程地址的后两位创建文件夹
- mysqldump 备份全库数据 (刷新binlog)
- 压缩备份数据
- 删除 14 天以外的备份数据

<a name="6bd39b9c"></a>
## 数据文件更换目录

用于将数据文件的目录迁移到别的挂载盘的目录:

```bash
#创建新的数据目录
mkdir -p /home/mysqldata/

#确认从库同步没有延迟后对主库进行read only 操作
set global read_only=1;

#停止主库和从库服务
service mysql stop

# 备份配置文件my.cnf
cp /etc/my.cnf /tmp

# 修改/etc/my.cnf中路径将所有的原路径改为新路径
# /path/to/src    为原路径 (mysql的数据、日志都在此目录下)
# /path/to/dest   为新路径
vim /etc/my.cnf
datadir=/home/mysqldata/

# 将源数据移动到新目录
cp -r /path/to/src /home/mysqldata/

# 更改新目录的所属用户
chown -R mysql:mysql /home/mysqldata/

# 启动主库和从库服务
service mysql start

# 从库执行：
start slave;
show slave status;
```

配套修改其他的数据库有关的组件, 例如备份脚本和监控平台中的路径.<br />一段时间后删除原来的数据可文件.

<a name="f30d6457"></a>
## 从生产远程同步数据到仿真环境

同样需要先创建对应同步数据的用户

- 远端用户需要 `SELECT` 和 `PROCESS` 权限
- 本地用户需要 `SELECT,INSERT,UPDATE,DELETE` 权限

```bash
vim syncDataFromProd.sh # 内容见下文

crontab -e              # 添加定时任务

# 添加如下记录，时间定在18点
0 18 * * * bash /root/syncDataFromProd.sh >> sync.log 2>&1
```

```bash
#!/usr/bin/bash

export PATH=$PATH:/usr/local/mysql/bin

echo "===== $(date): Synchronization start...  ====="

mysqldump -u用户名 -p密码    \
    数据库名 表名1 表名2 \
    --host=数据库IP          \
    --port=3306                 \
    --single-transaction=TRUE   \
    --compact                   \
    --no-create-info            \
    --complete-insert           \
    --skip-extended-insert      \
    --where="WHERE条件" > 数据库名.sql

mysql -u用户名 -p密码 -D数据库名 < 数据库名.sql
```

<a name="bf316264"></a>
## 常用脚本

导出MySQL指定表的记录（根据where子句）

```bash
mysqldump -u用户名 -p密码    \
    数据库名 表名1 表名2      \
    --host=IP地址           \
    --port=3306                 \
    --single-transaction=TRUE   \
    --compact                   \
    --no-create-info            \
    --complete-insert           \
    --skip-extended-insert      \
    --where="条件语句" > mysql_dump.sql
```

查看指定时间段MySQL binlog (MD5解码)

```bash
mysqlbinlog \
  --no-defaults binlog.000195 \
  --start-datetime='2021-06-07 00:00:00' \
  --stop-datetime='2021-06-08 00:00:00'  \
  --base64-output=decode-rows -vv > mysql_history_20210607.log
```


<a name="WaPB2"></a>
## 运维参数

计算缓存命中率:

```sql
show global status like 'innodb%read%';
```
![](https://cdn.nlark.com/yuque/__latex/8006547ca5422f8253214655e6bf236a.svg#card=math&code=Hit%5C%25%20%3D%20%5Cfrac%7BInnodb%5C_buffer%5C_pool%5C_read%5C_requests%7D%7BInnodb%5C_buffer%5C_pool%5C_read%5C_requests%20%2B%20Innodb%5C_buffer%5C_pool%5C_reads%20%2B%20Innodb%5C_buffer%5C_pool%5C_read%5C_ahead%7D&id=Ubgyw)

```
Hit% = Innodb_buffer_pool_read_requests/ (Innodb_buffer_pool_read_requests + Innodb_buffer_pool_reads + Innodb_buffer_pool_read_ahead)
```
