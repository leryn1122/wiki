<a name="zSc23"></a>
# RabbitMQ 安装手册
参考文档：

- [https://www.erlang.org/downloads](https://www.erlang.org/downloads)
- [https://www.rabbitmq.com/](https://www.rabbitmq.com/)
- [https://github.com/rabbitmq/rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)
<a name="OC3TN"></a>
## 安装 Erlang
```bash
wget -O- https://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc | sudo apt-key add -
echo "deb https://packages.erlang-solutions.com/ubuntu bionic contrib" | sudo tee /etc/apt/sources.list.d/rabbitmq.list
apt update & apt install -y erlang
```
<a name="yvF6a"></a>
## 安装 RabbitMQ
```bash
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.deb.sh | sudo bash
apt update & apt install rabbitmq-server
```
```bash
systemctl start  rabbitmq-server
systemctl status rabbitmq-server
systemctl enable rabbitmq-server
```
开启管理仪表盘：
```bash
rabbitmq-plugins enable rabbitmq_management
```
添加延迟队列插件：
```bash
mv rabbitmq_delayed_message_exchange-3.8.0.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.11/plugins/
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```
<a name="yPfke"></a>
## 搭建集群
配置主机名和本地域名解析：
```bash
# 设置主机名
hostnamectl set-hostname rabbitmq1
echo xxx.xxx.xxx.xxx rabbitmq1 >> /etc/hosts
echo xxx.xxx.xxx.xxx rabbitmq2 >> /etc/hosts
echo xxx.xxx.xxx.xxx rabbitmq3 >> /etc/hosts
```
要使 RabbitMQ 群集正常工作，参与群集的所有节点都应具有相同的 Cookie，将第一个节点上的 Cookie 复制到群集中的所有其他节点。
```bash
scp /var/lib/rabbitmq/.erlang.cookie rabbitmq2:/var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.cookie rabbitmq3:/var/lib/rabbitmq/.erlang.cookie
```
在各个**非主**节点上重置服务：
```bash
rabbitmqctl stop_app

rabbitmqctl reset

rabbitmqctl join_cluster rabbit@rabbitmq1

rabbitmqctl start_app

rabbitmqctl cluster_status
```
<a name="IPIRu"></a>
## 设置高可用策略
高可用可配置所有节点存储副本或 n 节点存储副本。在管理端的 Admin -> Policy 处可以看到。官方建议副本数设置为 （2/N + 1）。<br />根据测试结果，3 节点设置所有节点存储副本模式下，挂一台就处于不可用状态。
```bash
rabbitmqctl set_policy -p "/" \
  ha-two "^" '{"ha-mode":"exactly", "ha-params":2, "ha-sync-mode":"automatic"}'
```
