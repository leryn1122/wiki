<a name="VniWJ"></a>
# Mycat
参考文档:

- [Mycat 官网](http://www.mycat.org.cn/)
- [Mycat 权威指南 - Mycat 官网](http://www.mycat.org.cn/document/mycat-definitive-guide.pdf)
<a name="fNQaL"></a>
## 二进制安装
<a name="Thydt"></a>
### 前置准备

部署准备, 安装前需要准备如下材料:

- JDK 1.8, 以及对应的环境变量 JAVA_HOME 和 PATH.<br />较高的 JDK 版本不兼容一些默认的启动参数, 所以建议使用常规的 JDK 8 版本.<br />安装步骤参考: [JDK 安装手册](docs_java_jdk-deployment)
<a name="De7ia"></a>
### 安装步骤

- [MyCat 安装包下载](https://github.com/MyCATApache/Mycat-download)

```bash
wget https://github.com/MyCATApache/Mycat-download/blob/master/1.5-RELEASE/Mycat-server-1.5.1-RELEASE-20160811220036-linux.tar.gz
tar -xf Mycat-server-*-linux.tar.gz
sudo mv mycat /usr/local/mycat
```

<a name="tj0Y5"></a>
### 启动与验证

```bash
cd /usr/local/mycat/bin
./bin/mycat start
```
<a name="rN0EM"></a>
## Docker 安装

- [Docker 安装 Mycat - Mycat 官网](https://github.com/MyCATApache/Mycat-Server/wiki/2.1-docker%E5%AE%89%E8%A3%85Mycat)
- [Mycat 非官方镜像 - dockerhub](https://hub.docker.com/r/manondidi/mycat)

官网提供方案是自行按照操作步骤构建镜像, 这里偷懒使用第三方作者 build 的镜像.<br />以下步骤可以省略, 启动容器前创建挂载目录:

```bash
mkdir -p /conf/mycat
mkdir -p /logs/mycat

# 预先取出容器内的配置文件, 保留到容器外部
docker run \
  --detach=true \
  --publish=8066:8066 \
  --publish=9066:9066 \
  --name=mycat \
  --hostname=mycat \
  manondidi/mycat:1.6.7.5

docker cp mycat:/usr/local/mycat/conf/ /conf/mycat

cp -ar conf/* /conf/mycat && rm -rf conf/*
docker stop mycat && docker rm mycat
```

重新启动容器:

```bash
docker run \
  --detach=true \
  --publish=8066:8066 \
  --publish=9066:9066 \
  --volume=/conf/mycat:/usr/local/mycat/conf \
  --volume=/logs/mycat:/usr/local/mycat/logs \
  --name=mycat \
  --hostname=mycat \
  manondidi/mycat:1.6.7.5
```
