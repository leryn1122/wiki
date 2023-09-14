
# MySQL-Proxy 安装手册
MySQL Proxy 是调研 MySQL 高可用时, 发现的一个 MySQL 代理. 它是 MySQL 的官方实践, 但似乎还处于 **alpha** 版本.<br />它仅支持 MySQL 5.x, 如果是更高的版本, 请忽略这个中间件.<br />它中间使用了 Lua 脚本来管理节点以及读写分离, 据说延迟仅有 400ms, QPS 能到 2400+.

## 二进制安装
下载安装包并解压:
```bash
wget https://downloads.mysql.com/archives/get/p/21/file/mysql-proxy-0.8.5-linux-debian6.0-x86-64bit.tar.gz
tar -xf mysql-proxy-0.8.5-linux-debian6.0-x86-64bit.tar.gz
mv mysql-proxy-0.8.5-linux-debian6.0-x86-64bit /usr/local/mysql-proxy
```
修改配置文件, 这里我通过两个不同的端口来启动的两个 docker 上的 MySQL 容器, 一主一从:
```properties
[mysql-proxy]
admin-username=scott
admin-password=tiger
proxy-address=0.0.0.0:4040
proxy-read-only-backend-addresses=121.196.30.39:3307
proxy-backend-addresses=121.196.30.39:3306
proxy-lua-script=/usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua
admin-lua-script=/usr/local/mysql-proxy/share/doc/mysql-proxy/admin-sql.lua
log-file=/usr/local/mysql-proxy/logs/mysql-proxy.log
log-level=info
daemon=1
keepalive=1
```
创建日志文件:
```bash
mkdir -p /usr/local/mysql-proxy/logs
touch /usr/local/mysql-proxy/logs/mysql-proxy.log
```

## 启动与验证
启动 MySQL Proxy
```bash
bin/mysql-proxy --defaults-file=/etc/mysql-proxy.cnf
```
