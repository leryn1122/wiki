<a name="SrHc1"></a>
# Jfrog Artifactory 安装手册

参考文档:

- [https://www.jfrogchina.com/en/open-source/#artifactory](https://www.jfrogchina.com/en/open-source/#artifactory)
<a name="avrri"></a>
## 安装步骤
安装 JDK:
```bash
apt install -y openjdk-8-jre
```
安装 Jfrog Artifactory
```bash
mkdir -p /opt/jfrog
cd /opt/jfrog

# 
wget https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/[RELEASE]/jfrog-artifactory-oss-[RELEASE]-linux.tar.gz
tar -xf jfrog*.tar.gz

# 创建软连接
ARTIFACTORY_HOME=$(cd artifactory-oss* && pwd) && ln -sf $(basename $ARTIFACTORY_HOME) artifactory


cd /opt/jfrog/artifactory/app/

cat <<\EOF> /etc/profile.d/artifactory.sh
#!/usr/bin/env/bash

ARTIFACTORY_HOME=/opt/jfrog/artifactory/app
PATH="$ARTIFACTORY_HOME/bin":$PATH
EOF

source /etc/profile
```


<a name="M4oYt"></a>
## Docker 安装

`docker.bintray.io/jfrog/artifactory-oss:6.9.0` 带有 OSS 字样的都是社区版.<br />`dzaunerepos/jfrog-artifactory-pro:latest` 带有 Pro 的是专业版.

```nginx
docker run \
  --detach=true \
  --name artifactory \
  --hostname artifactory \
  --privileged \
  --publish=8081:8081 \
  --volume=/data/artifactory:/data/artifactory \
  dzaunerepos/jfrog-artifactory-pro

docker run \
  --detach=true \
  --name artifactory \
  --hostname artifactory \
  --publish=8081:8081 \
  --volume=/data/artifactory:/data/artifactory \
  dzaunerepos/jfrog-artifactory-pro

```
