<a name="u4O8M"></a>
# Elastic Search 安装手册
<a name="FeUce"></a>
## Debian 包管理器安装（推荐）
参考文档:

- [Elastic Search - Elastic 官网](https://www.elastic.co/cn/elasticsearch/)
- [Install Elasticsearch with Debian Package - Elastic 官网](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/deb.html#deb-repo)
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch kibana
```
修改配置文件
```bash
vim /etc/elasticsearch/elasticsearch.yml
```
```yaml
cluster.name: ES名称
#默认node-1, node-2, etc.

node.name: ES节点名称
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["集群IP地址1", "集群IP地址2", "集群IP地址3"]

# ES节点名称
cluster.initial_master_nodes: ["node-1"，"node-2", "node-3"]
```
（可选）修改启动内存限制，内存锁定需要配置 2g 以上的内存，否则会导致启动失败
```bash
vim /usr/lib/systemd/system/elasticsearch.service

LimitMEMLOCK=infinity
```
（可选）修改内存限制，这里是测试服务器所以只给2g
```bash
vim /etc/elasticsearch/jvm.options

-Xms2g
-Xmx2g
```
<a name="DAFaP"></a>
## 二进制安装（历史遗留不推荐）
ES不能在root用户上运行（在集群所有节点上新建用户和组）
```bash
groupadd elasticsearch
useradd elasticsearch -g elasticsearch -m -p 密码 -s /bin/bash
```
上传ES安装包，并解压到安装路径：
```bash
tar -xf elasticsearch-7.10.2-linux-x86_64.tar.gz -C /opt/ chown elasticsearch:elasticsearch /opt/elasticsearch-7.10.2 -R
```
进入安装目录，修改配置文件：
```yaml
#cp -ar config/elasticsearch.yml config/elasticsearch.yml.bak
#vim config/elasticsearch.yml

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
修改JVM参数：
```bash
#cp -ar config/jvm.options config/jvm.options.bak
#vim config/jvm.options

# 修改以下两个参数(两个参数必须一致，否则启动时会报错)
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space
-Xmx20g
-Xms20g
```
用`rsync`命令同步文件到全部节点上：
```bash
rsync -av 安装目录 root@IP地址:安装目录
```
逐个节点上修改配置文件中的节点名：
```bash
vim config/elasticsearch.yml
```
在`elasticsearch`用户家目录下的`~/.bashrc`末尾加上：
```bash
# Set ElasticSearch environment.
# ES安装路径
export ES_HOME=/opt/elasticsearch-7.10.2
export JAVA_HOME=${ES_HOME}/jdk
export PATH=$PATH:${ES_HOME}/bin:${JAVA_HOME}/bin
```
使环境变量生效：
```bash
source ~/.bashrc
```
设置系统的虚拟内存：
```bash
sysctl -w vm.max_map_count=262144
```
创建文件`start_es.sh`：
```bash
#!/bin/bash
elasticsearch -d
```
执行ES启动脚本：
```bash
./start_es.sh
```
同步脚本：
```bash
rsync -av /home/es/.bashrc root@IP地址:/home/es/
rsync -av /home/es/start_es.sh root@IP地址:/home/es/
```
官方似乎没有提供停止ES的命令，用`kill -9`的方式停止：
```bash
# 仅开发测试环境使用
ps -ef | grep elasticsearch | grep -v grep | awk '{print $2}' | xargs -r kill -9

# 生产上考虑安全性
ps -ef | grep elasticsearch
kill -9 PID
```
配置`systemctl`进行启动终止操作：
```bash
# 仅供参考
cd /etc/systemd/system
vim elasticsearch.service
```
```bash
# elasticsearch.service
[Unit]
Description=elasticsearch.service
After=network.target

[Service]
Type=forking
User=elasticsearch 
#使用这个账号操作
LimitNOFILE=100000
LimitNPROC=100000
ExecStart=/opt/elasticsearch-7.10.2/bin/elasticsearch -d

[Install]
WantedBy=multi-user.target
```
```bash
#参考http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html
#创建连接
systemctl enable  elasticsearch.service
#查看状态
systemctl status elasticsearch.service
#启动
systemctl start elasticsearch.service
#终止
systemctl stop elasticsearch.service
```
<a name="40ea745c"></a>
### Kibana 使用 nginx 配置权限认证
```bash
# 使用htpasswd 生成认证文件包含user passwd
apt install apache2-utils
mkidr -p /usr/local/nginx/conf/
htpasswd -bc /usr/local/nginx/conf/kibana.users kibana 123456

# 更改nginx 配置文件 在locaion添加 配置 保存 重启
auth_basic "请输入密码！";
auth_basic_user_file /usr/local/nginx/conf/kibana.users ;

nginx -s reload
```
