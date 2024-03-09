
# Confluence
参考文档：

- [Conflunce - 官网](https://www.atlassian.com/zh/software/confluence)

## Docker 安装

### Docker 镜像安装
官方镜像提供了镜像 [atlassian/confluence-server](https://hub.docker.com/r/atlassian/confluence-server)，提供了完善的配置和解决方案。
我们使用一个热门的第三方 [haxqer/confluence](https://hub.docker.com/r/haxqer/confluence)。
```bash
docker run \
  --detach=true \
  --env=TZ='Asia/Shanghai' \
  --publish=8090:8090 \
  --volume=/data/confluence:/var/confluence \
  --volume=/conf/confluence:/opt/confluence/conf \
  --name=confluence \
  --hostname=confluence \
  haxqer/confluence:latest
```
容器启动成功后进入对应的界面，根据界面提示操作即可。有两步需要额外的服务端操作。

### 生成序列号
如果需要生成序列号，可以使用如下命令生成序列号。
```bash
# -o 网址
# -s 服务器ID
docker exec confluence \
  java -jar /var/agent/atlassian-agent.jar \
  -p conf \
  -m haxqer666@gmail.com \
  -n haxqer666@gmail.com \
  -o https://confluence.mydomain.com \
  -s B0YY-DP3U-WMSM-9HVK
```

### 创建数据库
创建数据库，这里选择 MySQL：
注意 confluence 对字符集和事务隔离等级有要求。
创建数据库，注意要指定字符集`utf8`和排序字符集`utf8_bin`，其他都是常规操作。
```sql
CREATE USER confluence@'%' IDENTIFIED BY 'confluence@123' WITH GRANT OPTION;
CREATE DATABASE IF NOT EXISTS confluence DEFAULT CHARACTER SET utf8 COLLATE utf8_bin;
GRANT ALL PRIVILEGES ON confluence.* TO confluence@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
利用 JDBC URL 设置事务隔离等级。
```
jdbc:mysql://mysql.mydomain.com:3306/confluence?sessionVariables=transaction_isolation='READ-COMMITTED'&useSSL=false&allowPublicKeyRetrieval=true
```

### 修改 JVM 内存限制（可选）
默认的 JVM 内存限制最大最小内存都是 1024M。4G 贫民服务器需要降低内存才能正常工作。
```bash
docker exec confluence \
  sed -i 's/-Xms[0-9]*m -Xmx[0-9]*m/-Xms1024m -Xmx1024m/p' /opt/confluence/bin/setenv.sh
docker restart confluence
```

### 设置 Base URL
如果使用域名访问需要把 Base URL 设置成域名，而不是 IP 地址加端口的形式。或者再一开始生成序列号的时候就填写域名。
参考官方文档：

- [更多关于服务器基础 URL - 官方文档](https://confluence.atlassian.com/conf76/configuring-the-server-base-url-1018769760.html)
- [在 Confluence 6.6 或更高版本中无法检查基本 URL 警告 - 官方文档](https://confluence.atlassian.com/confkb/can-t-check-base-url-warning-in-confluence-6-6-or-later-939718433.html)

站点管理 -> 一般配置 -> 一般配置 -> 服务器主页 URL 修改为`[https://<subdomain>.<domain>.com](https://%3Csubdomain%3E.%3Cdomain%3E.com)`并提交。同时修改`**${CONF_HOME}**/conf/server.xml`的配置文件，
```bash
docker cp confluence:/opt/confluence/conf/server.xml .

vim server.xml
```
注释以下部分：
```xml
<Connector port="8090" connectionTimeout="20000" redirectPort="8443"
           maxThreads="48" minSpareThreads="10"
           enableLookups="false" acceptCount="10" debug="0" URIEncoding="UTF-8"
           protocol="org.apache.coyote.http11.Http11NioProtocol"/>
```
并取消注释以下部分，并修改 Base URL：
```xml
<Connector port="8090" connectionTimeout="20000" redirectPort="8443"
           maxThreads="48" minSpareThreads="10"
           enableLookups="false" acceptCount="10" debug="0" URIEncoding="UTF-8"
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           scheme="https" secure="true" proxyName="<subdomain>.<domain>.com" proxyPort="443"/>
```
保存并重启 Confluence：
```bash
docker cp ./server.xml confluence:/opt/confluence/conf/server.xml
docker restart confluence
```

## 二进制安装
参考文档：

- [Confluence Server 7.6 Archive 方式安装文档 - Atlassian 官网](https://confluence.atlassian.com/conf76/installing-confluence-on-linux-from-archive-file-1018769978.html)
```bash
前置准备

● Confluence: 7.6.2, 绑定的 Tomcat 版本 9.0.33
● mysql: 5.7.36
● JDK版本: OpenJDK 8u202

二. 安装JDK

1.  OpenJDK download: https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u202-b08/OpenJDK8U-jdk_x64_linux_hotspot_8u202b08.tar.gz 
2.  解压缩到/usr/local/java/ 
3.  配置JAVA环境变量: `/etc/profile.d/java.sh`
export JAVA_HOME=/usr/local/java/jdk8u202-b08
export CLASS_PATH=$JAVA_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin
三. 安装MySQL 
4.  mysql 5.7.36 download: https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.36-el7-x86_64.tar.gz 
5.  解压缩到/usr/local/mysql 
6.  配置mysql环境变量: /etc/profile.d/mysql.sh
export PATH=$PATH:/usr/local/mysql/bin 
7.  建立mysql用户以及数据文件目录：https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html
$> groupadd mysql
$> useradd -r -g mysql -s /bin/false mysql
$> mkdir /data/mysql
$> chown mysql:mysql /data/mysql
$> chmod 750 mysql-files 
8.  配置/etc/my.cnf: https://confluence.atlassian.com/conf76/database-setup-for-mysql-1018769687.html


[mysqld]
...
datadir=/data/mysql
basedir=/usr/local/mysql
character-set-server=utf8mb4
collation-server=utf8mb4_bin
default-storage-engine=INNODB
max_allowed_packet=256M
innodb_log_file_size=2GB
transaction-isolation=READ-COMMITTED
binlog_format=row
... 

9.  初始化数据库,启动MySQL：

bin/mysqld --initialize --user=mysql
bin/mysql_ssl_rsa_setup
bin/mysqld_safe --user=mysql &

可选：重置root密码，建立confluence数据库账户


10.  创建confluence数据库：https://confluence.atlassian.com/conf76/database-setup-for-mysql-1018769687.html

CREATE DATABASE confluence CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
GRANT ALL PRIVILEGES ON confluence .* TO ''@'localhost' IDENTIFIED BY ''; 

11.  创建目录： 

/opt/confluence: confluence安装路径

/var/confluence：confluence运行时路径

/var/agent: 破解jar包路径

2.  下载confluence 压缩包, 并解压到/opt/confluence：https://product-downloads.atlassian.com/software/confluence/downloads/atlassian-confluence-7.6.2.tar.gz 
3.  解压缩：
tar xzf /tmp/atlassian-confluence-7.6.2.tar.gz -C /opt/confluence/ --strip-components 1 
4.  解压缩并复制mysql JDBC Driver 到 /opt/confluence/confluence/WEB-INF/lib: https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-5.1.48.tar.gz 
5.  下载破解包到 /var/agent: https://github.com/haxqer/confluence/releases/download/v1.2.2/atlassian-agent.jar 
6.  编辑/opt/confluence/confluence/WEB-INF/classes/confluence-init.properties, 指定confluence运行时路径
confluence.home=/var/confluence 
7.  编辑tomcat 环境变量启动脚本，注入破解包
/opt/confluence/bin/setenv.sh
line 67: CATALINA_OPTS="-javaagent:/var/agent/atlassian-agent.jar ${CATALINA_OPTS}" 
8.  启动confluence：
/opt/confluence/bin/start-confluence.sh
初始化启动之前建议打快照 
9.  登录到confluence 进行初始化：https://confluence.atlassian.com/conf76/installing-confluence-on-linux-from-archive-file-1018769978.html 

生成序列号

java -jar /var/agent/atlassian-agent.jar -p conf \
  -m haxqer666@gmail.com -n haxqer666@gmail.com \
  -o https://confluence.mydomain.com -s <your system id>

注意每次初始化system id均不同, 若初始化过程中关闭confluence进程会导致confluence再次启动失败

五. 配置Confluence

1. 配置Tomcat 默认Server Address: https://confluence.atlassian.com/confkb/can-t-check-base-url-warning-in-confluence-6-6-or-later-939718433.html

根据反向代理编辑 /opt/confluence/conf/server.xml

2.  因为confluence服务器关闭了外网访问，在一般配置中，关闭Confluence Atlassian Marketplace功能 
3.  将Confluence 认证方式与JIRA 链接起来： https://confluence.atlassian.com/conf76/connecting-to-crowd-or-jira-for-user-management-1018769563.html 

首先配置允许访问JIRA User Server的application

在Confluence 新增JIRA类型的User Directories

注意JIRA Server URL后加端口

4.  配置日志轮替
$>cat /etc/logrotate.d/confluence
/opt/confluence/logs/catalina.out
/var/confluence/logs/atlassian-confluence.log
/var/confluence/logs/atlassian-synchrony.log
/var/confluence/logs/atlassian-diagnostics.log
{
missingok
daily
rotate 14
missingok
dateext
notifempty
copytruncate
} 
```

## 集成 Jira
参考官方文档：

- [在设置向导中配置 Jira 集成 - 官方文档](https://confluence.atlassian.com/doc/configuring-jira-integration-in-the-setup-wizard-242255467.html)

需要 Jira 管理员权限，根据提示操作。

## 备份与恢复
参考官方文档：

- [生产备份策略 - 官方文档](https://confluence.atlassian.com/conf76/production-backup-strategy-1018769697.html)
- [手动备份站点 - 官方文档](https://confluence.atlassian.com/conf76/migrating-to-another-database-1018769689.html)

虽然 Confluence 提供了预定的 XML 备份，但这种备份方式只适用于**小型站点**，以及除了数据库和**目录**备份**之外**的备份。

### 数据库备份策略
我们建议建立一个健壮的数据库备份策略：

- 使用数据库提供的工具创建数据库的备份或转储
- 如果您的数据库不支持在线备份，您需要在执行此操作时停止 Confluence
- 创建主目录的文件系统备份（本地主目录和数据中心的共享主目录）

### 需要备份的文件
备份整个主目录是最安全的选择，但是大多数文件和目录在启动时填充，可以忽略。至少必须备份这些文件/目录：
以下 `${CONF_HOME}` 为 confluence 数据的根目录，当前 docker 容器中为`/data/cofluence:/var/confluence`。

- `**${CONF_HOME}/confluence.cfg.xml**`配置文件
- `**${CONF_HOME}/attachment**`附件

其余目录将在启动时自动填充。您可能还想备份这些目录：

- `**${CONF_HOME}/config**` - 如果您修改了 `ehcache.xml` 文件
- `**${CONF_HOME}/index**` - 如果您的站点很大或重新索引需要很长时间 - 这将避免在恢复时需要完整的重新索引。主目录的位置在安装时配置并在 `confluence.init.properties` 文件中指定。对于使用自动安装程序创建的安装, 默认位置为：
   - Windows: `C:\Program Files\Atlassian\Application Data\Confluence`
   - Linux: `/var/atlassian/application-data/confluence`

仅适用于集群实例：备份整个共享主目录是最安全的选择，但是一些文件和目录在运行时填充并且可以忽略：

- `**${CONF_HOME}/thumbnails**`
- `**${CONF_HOME}/viewfile**`

### 备份

- [生产备份策略 - 官方文档](https://confluence.atlassian.com/conf76/production-backup-strategy-1018769697.html)

### 从备份恢复

- [手动备份站点 - 官方文档](https://confluence.atlassian.com/conf76/migrating-to-another-database-1018769689.html)
