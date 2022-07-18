
![](https://s3.leryn.top/website/image/maven.svg#crop=0&crop=0&crop=1&crop=1&height=63&id=UnGAE&originHeight=86&originWidth=340&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=250)
<a name="oB6Sm"></a>
# Maven 安装手册
<a name="sIIXd"></a>
## 包管理器

很简单一句命令即可, 但是可能缺少 protobuff 并无法使用
<a name="uaNNi"></a>
### 安装步骤

```bash
sudo apt install -y maven
```
<a name="yfFwF"></a>
## 二进制安装
<a name="HdVOW"></a>
### 前置准备

下载二进制安装包:

```bash
wget https://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz
```
<a name="kIrrm"></a>
### 安装步骤

解压安装包到指定路径, 并配置环境变量

```
tar -xf apache-maven-3.8.4-bin.tar.gz -C /path/to/maven
```
```bash
cat <<EOF | sudo tee /etc/profile.d/env-mvn.sh
#!/usr/bin/env bash
# Maven environment.
export MAVEN_HOME=/path/to/maven
export M2_HOME=${}
export PATH=${MAVEN_HOME}/bin;${PATH}
```

通常都会再修改默认的 maven 配置, 位置在`${MAVEN_HOME}/conf/settings.xml`
<a name="Ae05Y"></a>
### 启动与验证

```bash
mvn -v
```

```
Apache Maven 3.8.4 (9b656c72d54e5bacbed989b64718c159fe39b537)
Maven home: /opt/module/apache-maven-3.8.4
Java version: 17, vendor: Private Build, runtime: /usr/lib/jvm/java-17-openjdk-amd64
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "5.4.0-77-generic", arch: "amd64", family: "unix"
```
<a name="M5WkK"></a>
## Docker

```bash
docker pull maven:3.8.4-openjdk-17-slim
docker pull maven:3.8.4-jdk-8-slim
```
<a name="KIvi1"></a>
# Maven-Daemon

`mvnd` 是 apache/maven 的一个子项目, 它并不是一个全新的构建工具, 而是对 maven 的扩展. 它内置了 maven, 其实现原理是构建了一个或者多个 maven 守护进程来执行构建服务.

1. mvnd 的目标是使用 Gradle 和 Takari 所知的技术提供更快的 Maven 构建. Gradle 是一个基于 Apache Ant 和 Apache Maven 概念的项目自动化建构工具. Gradle 构建脚本使用的是 Groovy 或 Kotlin 的特定领域语言来编写的, 而不是传统的 XML. Gradle 最大的优势就是比传统的 Maven 构建速度更快. Takari 是 maven-wrapper 核心, 大部分的开源项目都是提供 warpper 方便用户不安装 maven 的前提下快速构建项目的. 
1. mvnd 内嵌了 Maven, 所以不需要单独安装 Maven 了. 
1. 一个守护进程实例可以服务于来自 mvnd 客户端的多个连续请求. 
1. mvnd 客户端使用 GraalVM 构建本地可执行文件, 与启动传统 JVM 相比, 它启动得更快, 占用的内存更少. 
1. 如果 mvnd 没有空闲守护进程来服务一个构建请求, 可以并行地生成多个守护进程.

<a name="le1XO"></a>
## 二进制安装
<a name="OwkQt"></a>
## 构建镜像
目前还没有官方镜像, 但是目前构建完镜像使用仍然有点问题. 构建镜像, 需要以下文件:

- `Dockerfile`
- `settings-docker.xml`
- `mvn-entrypoint.sh`
<a name="e4xs2"></a>
### `Dockerfile`
```dockerfile
FROM docker.leryn.top/openjdk:17-oracle
ARG MAVEN_VERSION=0.7.1
ARG USER_HOME_DIR="/root"
ARG SHA=ac0b276d4d7472d042ddaf3ad46170e5fcb9350981af91af6c5c13e602a07393
#ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries
ARG BASE_URL=https://download.leryn.top/release/
RUN microdnf install unzip findutils git \
  && microdnf clean all
COPY mvnd-${MAVEN_VERSION}-linux-amd64.zip /tmp/mvnd-linux-amd64.zip
RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && echo "${SHA}  /tmp/mvnd-linux-amd64.zip" | sha256sum -c - \
  && unzip /tmp/mvnd-linux-amd64.zip -d /tmp \
  && mv /tmp/mvnd-${MAVEN_VERSION}-linux-amd64/* /usr/share/maven \
  && rm -rf /tmp/mvnd-linux-amd64* \
  && ln -s /usr/share/maven/bin/mvnd /usr/bin/mvnd
  # && curl -fsSL -o /tmp/mvnd-linux-amd64.zip ${BASE_URL}/mvnd-${MAVEN_VERSION}-linux-amd64.zip \
ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"
COPY mvn-entrypoint.sh /usr/local/bin/mvn-entrypoint.sh
COPY settings-docker.xml /usr/share/maven/ref/
ENTRYPOINT ["/usr/local/bin/mvn-entrypoint.sh"]
CMD ["mvnd"]
```
<a name="SV7KN"></a>
### `settings-docker.xml`
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                            https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>/usr/share/maven/ref/repository</localRepository>
</settings>
```
<a name="buKG3"></a>
### `mvn-entrypoint.sh`
```bash
#! /bin/sh -eu
# Copy files from /usr/share/maven/ref into ${MAVEN_CONFIG}
# So the initial ~/.m2 is set with expected content.
# Don't override, as this is just a reference setup
copy_reference_files() {
  local log="$MAVEN_CONFIG/copy_reference_file.log"
  local ref="/usr/share/maven/ref"
  if mkdir -p "${MAVEN_CONFIG}/repository" && touch "${log}" > /dev/null 2>&1 ; then
      cd "${ref}"
      local reflink=""
      if cp --help 2>&1 | grep -q reflink ; then
          reflink="--reflink=auto"
      fi
      if [ -n "$(find "${MAVEN_CONFIG}/repository" -maxdepth 0 -type d -empty 2>/dev/null)" ] ; then
          # destination is empty...
          echo "--- Copying all files to ${MAVEN_CONFIG} at $(date)" >> "${log}"
          cp -rv ${reflink} . "${MAVEN_CONFIG}" >> "${log}"
      else
          # destination is non-empty, copy file-by-file
          echo "--- Copying individual files to ${MAVEN_CONFIG} at $(date)" >> "${log}"
          find . -type f -exec sh -eu -c '
              log="${1}"
              shift
              reflink="${1}"
              shift
              for f in "$@" ; do
                  if [ ! -e "${MAVEN_CONFIG}/${f}" ] || [ -e "${f}.override" ] ; then
                      mkdir -p "${MAVEN_CONFIG}/$(dirname "${f}")"
                      cp -rv ${reflink} "${f}" "${MAVEN_CONFIG}/${f}" >> "${log}"
                  fi
              done
          ' _ "${log}" "${reflink}" {} +
      fi
      echo >> "${log}"
  else
    echo "Can not write to ${log}. Wrong volume permissions? Carrying on ..."
  fi
}
owd="$(pwd)"
copy_reference_files
unset MAVEN_CONFIG
cd "${owd}"
unset owd
exec "$@"
```
