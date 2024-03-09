
# MySQL 升级方案
参考文档:

- [https://dev.mysql.com/doc/relnotes/mysql/8.0/en/](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/)

## 禁用预编译语句

1. 检查jdbc连接，如有设置 `useServerPrepStmts=true`，将其改为 `false`。重启应用。（所需时间最短）

JDBC 连接将绑定变量功能开启：数据库 general_log 显示 prepare 字样，就表示使用了绑定变量
JDBC 连接将绑定变量功能关闭：数据库 general_log 不会显示 prepare 字样，就表示没有使用绑定变量

## 对 MySQL 就地升级

1. 对原 mysql 进行备份，并将备份拷贝到远程服务器，以防升级误操作删除备份。
```bash
mysqldump -u root -p \
  --all-databases \
  --single-transaction \
  --master-data=2 \
  --triggers \
  --routines \
  --events > full_database_20220425.sql 
```

2. 升级前检测
```bash
/usr/local/mysql/bin/mysqlcheck -u root -p --all-databases --check-upgrade
```

3. 关闭数据库

关闭快速关闭数据库，让所有数据在关机时都写入硬盘。
```sql
set global innodb_fast_shutdown = 0;
shutdown
```

4. 解压最新 MySQL 安装包到 `/usr/local`
```sql
tar -xvf mysql-8.0.28-linux-glibc2.12-x86_64.tar.xz -C /usr/local 
```

5. 将mysql软链接指向新的 MySQL 目录
```bash
cd /usr/local
unlink mysql
ln -s mysql-8.0.28-linux-glibc2.12-x86_64 mysql
```

6. 启动数据库，在这时候默认就开始升级数据库了，注意实时观察 error.log，如没有问题，连接应用测试。
```bash
tail -f error.log
/usr/local/mysql/bin/mysqld_safe --user=mysql &
```

7. 检查版本
```sql
select version();
```

## 重新安装 MySQL

1. 对原 mysql 进行备份，并将备份拷贝到远程服务器，以防升级误操作删除备份。
```bash
mysqldump -u root -p \
  --all-databases \
  --single-transaction \
  --master-data=2 \
  --triggers \
  --routines \
  --events > full_database_20220425.sql 
```

2. 原始数据库软件和数据
```bash
rm -rf /opt/mydata /usr/local/mysql 
```

3. 安装新版本数据库
4. 解压最新 MySQL 安装包到 `/usr/local`
```sql
tar -xvf mysql-8.0.28-linux-glibc2.12-x86_64.tar.xz -C /usr/local 
```

5. 将数据库还原，注意实时观察 error.log
```bash
tail -f error.log
mysql -u root -p --force < full_database_20220425.sql
```

6. 重启数据库，并执行升级操作，注意实时观察 error.log
```bash
tail -f error.log
mysqladmin -u root -p shutdown
mysqld_safe --user=mysql --upgrade=FORCE & 
```

7. 检查版本
```sql
select version();
```

