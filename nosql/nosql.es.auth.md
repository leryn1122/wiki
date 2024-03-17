---
id: nosql.es.auth
tags: []
title: "Elasticsearch \u8BA4\u8BC1\u529F\u80FD"

---


# Elasticsearch 认证功能


## 修改 Elasticsearch 配置


### 新增配置
编辑 `elasticsearch.yml` 文件, 每个集群节点都需要设置:
```bash
vim /etc/elasticsearch/elasticsearch.yml
```
追加如下内容:
```yaml
xpack.security.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```


### 生成 TLS 和身份验证
执行下面操作生成证书:
```bash
/usr/share/elasticsearch/bin/elasticsearch-certutil cert -out /etc/elasticsearch/elastic-certificates.p12 -pass ""
```
将节点 1 上的证书依次拷贝到其他节点，**不要在各个节点上独自生成**。


### 重启 ES 集群
Elasticsearch 集群不重新启动，下面的添加密码操作执行不了，所以依次重启所有节点：
```bash
systemctl restart elasticsearch
```


### 创建 Elasticsearch 集群密码
在一个节点上执行如下命令，设置用户密码。设置完之后, 数据会自动同步到其他节点：
```bash
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```
会出现如下内容:
```bash
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]: 
Reenter password for [elastic]: 
Enter password for [apm_system]: 
Reenter password for [apm_system]: 
Enter password for [kibana]: 
Reenter password for [kibana]: 
Enter password for [logstash_system]: 
Reenter password for [logstash_system]: 
Enter password for [beats_system]: 
Reenter password for [beats_system]: 
Enter password for [remote_monitoring_user]: 
Reenter password for [remote_monitoring_user]: 
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```
依次填写上面系统的密码, 记住 Kibana 系统的名称（可能是`kibana_system`），后面修改 Kibana 配置要用到。


### 访问验证
执行完上述操作后就已经加上了密码，此时直接 curl 会报错：
```bash
curl -XGET http://localhost:9200
```
带上用户信息 curl 可以正常返回 Elasticsearch 信息：
```bash
curl --user elastic http://localhost:9200
```


## 修改 Kibana 配置
如果 Elasticsearch 同时配套使用了 Kibana 的话，请修改`kibana.yml`文件:
```bash
vim /usr/share/kibana/config/kibana.yml
```
添加上 Elasticsearch 集群的用户名和密码:
```yaml
elasticsearch.username: "kibana_system"
elasticsearch.password: "密码"
```
重启 Kibana 即可：
```bash
systemctl restart kibana
```
