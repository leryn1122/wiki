---
id: nosql.elastic.install
tags: []
title: "Elasticsearch \u5B89\u88C5\u624B\u518C"

---
# ElasticSearch 安装手册
## Debian 包管理器安装（推荐）
参考文档:

+ [Elasticsearch：官方分布式搜索和分析引擎 | Elastic](https://www.elastic.co/cn/elasticsearch/)
+ [Install Elasticsearch with Debian Package | Elasticsearch Guide [7.3] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/deb.html#deb-repo)

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

（可选）修改内存限制，这里是测试服务器所以只给 2g

```bash
vim /etc/elasticsearch/jvm.options

-Xms2g
-Xmx2g
```

