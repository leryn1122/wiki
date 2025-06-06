---
id: mq.kafka
tags:
- kafka
- mq
title: Kafka

---
# Kafka
参考文档：

+ [https://kafka.apache.org](https://kafka.apache.org)

安装步骤：

## Docker 安装
**<font style="color:#DF2A3F;">以下是两点更新：</font>**

+ 随着 Kafka 3.x 发布，不需要依赖外部的 Zookeeper 了
+ 转用 `bitnami/kafka:3.4` 官方的镜像

很遗憾，Kafka 没有官方镜像，以下是推荐的镜像：

> **bitnami/kafka:3.4**
>
> bitnami 的最新镜像，需要外部 Zookeeper
>
> **wurstmeister/kafka**
>
> 只包含了 Kafka，因此需要另行提供 ZooKeeper，推荐使用同一作者提交的 wurstmeister/zookeeper
>
> **landoop/fast-data-dev**
>
> 提供了一整套包括 Kafka、ZooKeeper、Schema Registry、Kafka-Connect 等在内的多种开发工具和 Web UI 监视系统。基本上是我见过的最强大的开发环境，尤其是对于 Kafka。
>
> Connect 的支持，包含了 MongoDB，ElasticSearch，Twitter 等超过 20 种 Connector，并且提供了通过 REST API 提交 Connector 配置的 Web UI。
>

选择 `bitnami/kafka:3.4`的镜像，不需要外部 Zookeeper

```bash
docker run -itd \
   --publish=9092:9092 \
   --env=ALLOW_PLAINTEXT_LISTENER=yes \
   --volume=/data/kafka:/bitnami \
   docker.io/bitnami/kafka:3.4
```

这里选择`wurstmeister/kafka`镜像，这里假设已经拥有了一个可用的外部 Zookeeper:

```bash
docker run \
  --detach=true \
  --env=KAFKA_ADVERTISED_HOST_NAME=localhost \
  --env=KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://xxx.xxx.xxx.xxx:9092 \
  --env=KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
  --env=KAFKA_ZOOKEEPER_CONNECT=xxx.xxx.xxx.xxx:2181 \
  --publish=9092:9092 \
  --volume=/data/kafka/:/kafka/kafka-logs-kafka \
  --name=kafka \
  --hostname=kafka \
  wurstmeister/kafka:2.13-2.8.1
```

## 二进制安装
### 前置准备
部署准备，安装前需要准备如下材料：

+ JRE 环境：至少 Java 8
+ ZooKeeper 环境，可以正常使用的 Zookeeper 环境即可，当然也可以使用 Kafka 内置的 ZooKeeper。2.8 以上的版本，Kafka 使用了 Raft 机制可以不使用 ZooKeeper 作为注册中心，当然新特性还不完全稳定
+ Kafka 安装包，安装包上的两个数字前者表示 Scala 的版本，后者表示 Kafka 自身的版本。官方推荐 Scala 的版本为 2.13

下载 Kafka 的 tar 包：

```bash
wget https://www.apache.org/dyn/closer.cgi?path=/kafka/2.7.0/kafka_2.13-2.7.0.tgz
wget https://mirrors.bfsu.edu.cn/apache/kafka/2.7.0/kafka_2.13-2.7.0.tgz
```

### 安装步骤
配置环境变量，并使其生效：

```bash
#!/usr/bin/env bash
# Set Kafka environment.
export KAFKA_HOME=/opt/module/kafka-2.7.0
export PATH=${PATH}:${KAFKA_HOME}/bin
```

安装 Kafka：

```bash
SCALA_VERSION=2.13
VERSION=2.7.0
tar -xf kafka_${SCALA_VERSION}-${VERSION}.tgz
mv kafka_${SCALA_VERSION}-${VERSION} /opt/module/
```

修改配置文件：

默认配置即可，需要注意的是必须配置 Kafka 和 ZooKeeper 监听的地址，且必须是实际路由地址，不能是`0.0.0.0`。如果需要暴露对外服务，请填写公网 IP 地址。

```properties
# The address the socket server listens on. It will get the value returned from
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092
listeners=PLAINTEXT://localhost:9092
# A comma separated list of directories under which to store log files
log.dirs=/tmp/kafka-logs
# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181
```

### 启动与验证
启动 Kafka：与其他的中间件不同，Kafka 必须要显示的指定配置文件才能正常启动：

```bash
# 启动Kafka
bin/kafka-server-start.sh -daemon config/server.properties
# 停止Kafka
bin/kafka-server-stop.sh
```

验证 Kafka 安装是否成功：

官方提供了控制台上调试的接口

```bash
# 创建topic
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test_jmx --partitions 1 --replication-factor 1
# 进入生产者, 发送测试消息
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_jmx
# 另一个会话中, 开启消费者, 接受消息
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_jmx --from-beginning
```

## Systemd
生产通常实体机部署而非 Docker，自己写了 systemd 文件：

```toml
[Unit]
Description=ZooKeeper-3.6.3 service
Wants=network-online.target
After=syslog.target
After=network.target

[Service]
Type=forking
Environment=ZOO_LOG_DIR=/data/zookeeper/logs
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/jdk1.8.0_231/bin"
User=root
Group=root
ExecStart=/opt/apache-zookeeper/bin/zkServer.sh start /opt/apache-zookeeper/conf/zoo.cfg
ExecStop=/opt/apache-zookeeper/bin/zkServer.sh stop /opt/apache-zookeeper/conf/zoo.cfg
ExecReload=/opt/apache-zookeeper/bin/zkServer.sh restart /opt/apache-zookeeper/conf/zoo.cfg

[Install]
WantedBy=multi-user.target
```

```toml
[Unit]
Description=Apache-Kafka-2.8.1 service 
After=network.target
After=zookeeper.service

[Service]
Type=simple
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/jdk1.8.0_231/bin"
Environment="KAFKA_OPTS=-javaagent:/opt/kafka/libs/jmx_prometheus_javaagent-0.16.1.jar=19092:/opt/kafka/config/jmx_exporter.yml"
User=root
Group=root
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

# Kafka SDK
## Springboot 支持
使用 Springboot 默认依赖即可

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka -->
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
  <version>3.3.6</version>
</dependency>
```

配置文件和手动配置 Bean

```java
@Configuration
public class KafkaConfiguration {
    
    @Bean
    public KafkaAdmin kafkaAdmin(KafkaProperties props) {
        Map<String, Object> config = new HashMap<>();
        config.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, props.getBootstrapServers());
        return new KafkaAdmin(config);
    }
    
    @Bean
    public NewTopic topic() {
        return TopicBuilder.name("topic_name")
                .partitions(10)
                .replicas(2)
                .build();
    }
    
}
```

创建生产者

```java
@Configuration
public class KafkaConfiguration {

    @Bean
    public ProducerFactory<String, String> producerFactory(KafkaProperties props) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, props.getBootstrapServers());
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(configs);
    }
    
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate(ProducerFactory<String, String> producerFactory) {
        return new KafkaTemplate<>(producerFactory);
    }
}
```

创建消费者

```java
@Configuration
public class KafkaConfiguration {

    @Bean
    public ConsumerFactory<String, String> consumerFactory(KafkaProperties props) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, props.getBootstrapServers());
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(configs);
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory(
        ConsumerFactory<String, String> consumerFactory
    ) {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory);
        return factory;
    }
    
    @KafkaListener(
        topics = {
            "topic_name"
        },
        topicPartitions = @TopicPartition(
            topic = "topic_name",
            partitionOffsets = {
                @PartitionOffset(partition = "0", initialOffset = "0"),
            }
        ),
        groupId = "consumer_group_id" 
    )
    public void listenToKafka(String message) {
        System.out.println(message);
    }
    
}
```

可以过滤特定条件的 Message 消费

```java
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory(
        ConsumerFactory<String, String> consumerFactory
    ) {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory);
        factory.setRecordFilterStrategy(
            record -> record.value().contains("message+flags")
        );
        return factory;
    }
```

自定义化序列器

```java
    @Bean
    public ProducerFactory<String, Object> producerFactory(KafkaProperties props) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, props.getBootstrapServers());
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        configs.put(ProducerConfig.TYPE_MAPPINGS,
                "vehicle:io.github.leryn.wiki.entity.message.Vehicle, " +
                "camera:io.github.leryn.wiki.entity.message.camera"
                );
        return new DefaultKafkaProducerFactory<>(configs);
    }
```

# Kafka Exporter
参考文档：

+ [GitHub - danielqsj/kafka_exporter: Kafka exporter for Prometheus](https://github.com/danielqsj/kafka_exporter)
+ [Release 1.4.2 / 2021-09-16 · danielqsj/kafka_exporter](https://github.com/danielqsj/kafka_exporter/releases/tag/v1.4.2)

推荐 `jmx_exporter` 并用

+ `jmx_exporter` 监控 jvm
+ `kafka_exporter` 监控 kafka，尤其是这个 exporter 有一个指标表征了 consumer、topic、partition 对应的 offset 和 lag

## Systemd 文件
`/etc/kafka_exporter/kafka_exporter.conf` 写好所有的配置：

```yaml
OPTIONS=" --kafka.server=localhost:9092 --kafka.server=... "
```

```toml
[Unit]
Description=Kafka Exporter Service
After=network.target
After=kafka.service

[Service]
Type=simple
User=root
Group=root
EnvironmentFile=-/etc/kafka_exporter/kafka_exporter.conf
ExecStart=/opt/kafka_exporter/kafka_exporter $OPTIONS

[Install]
WantedBy=multi-user.target
```

# KafkaTools 安装手册
+ [Offset Explorer](https://www.kafkatool.com/download.html)

下载对应版本，安装即可。页面如下：

![](./../assets/1649832827449-1fb072b7-5c6b-4944-9f40-1ea94e2e7fcb.png)


