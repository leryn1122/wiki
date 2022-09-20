<a name="ZrfhU"></a>
# JDK 安装手册

参考文档:

- [Oracle JDK - 官网](https://www.oracle.com/java/)
- [OpenJDK - 官网](https://openjdk.java.net/)

写在前面: 本仓库都不会区分 JDK 和 JRE 的区别.
<a name="yOerK"></a>
## 包管理器

```bash
sudo apt install openjdk-8-jdk
sudo apt install openjdk-11-jdk
sudo apt install openjdk-17-jdk
```

验证 JDK 是否安装成功, 使用命令查看版本, 如果可以正确显示版本信息则安装成功.

```bash
java -version
```

如果需要在多版本间切换.

```bash
# 根据提示切换版本, 输入版本前的编号
sudo update-alternatives --config java
```

验证 JDK 是否安装成功, 使用命令查看版本, 如果可以正确显示版本信息则安装成功.

```bash
java -version
```
<a name="yMAvo"></a>
## 环境变量

可能需要配置环境变量(一旦配置了切换版本会比较麻烦). 有些版本可能需要额外指定 JRE 路径, 这里暂时没发现不配置的问题.<br />`JAVA_HOME`指向 JDK 安装目录, `PATH`指向`java`命令所在的目录, 参考[JDK 环境变量](https://www.yuque.com/attachments/yuque/0/2022/sh/26002940/1643083448140-5139c455-2ed3-40b8-9360-eb1382fdcfba.sh).

```bash
cat <<EOF | sudo tee /etc/profile.d/env-jdk.sh
#!/usr/bin/env bash
# Set JDK Environment.
export JAVA_VERSION=8
export JAVA_HOME=/usr/lib/jvm/java-${JAVA_VERSION}-openjdk-amd64
export PATH=${JAVA_HOME}/bin:$PATH
EOF
```
