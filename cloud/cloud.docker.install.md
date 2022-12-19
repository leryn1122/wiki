<a name="mGk0h"></a>
# Docker 安装手册

十分推荐使用 Ubuntu 系统安装 Docker, 而不是 CentOS. 这基于以下原因: 

- Linux 内核会为 containerd 让路并提供相应的支持, 而 CentOS 的发行版的更新速度完全跟不上变化速度;
- 相对比较多的开发者使用 Ubuntu 来作为开发机或开发环境的构建.
<a name="P87BF"></a>
## 包管理器
<a name="rNTp5"></a>
### 安装步骤
```bash
apt update && apt install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"

apt update && apt install -y \
  containerd.io docker-ce docker-ce-cli

cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
    "https://hub-mirror.c.163.com",
    "https://harbor.leryn.top"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "features": {
    "buildkit": true
  }
}
EOF
```
<a name="C3N6q"></a>
#### 
<a name="XH88e"></a>
#### 开启 docker 远程调试端口

在 systemctl 的文件中配置端口, 追加一句 `-H tcp://0.0.0.0:12375`(目前我的 docker 服务尚未配置 TLS 安全证书):

```bash
sudo vim /usr/lib/systemd/system/docker.service
```

```bash
# 由于公网上默认端口2375比较容易收到攻击, 换个12375端口
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:12375
```

重启 docker 服务.

```bash
systemctl daemon-reload
systemctl restart docker
```

<a name="IsKlC"></a>
### 启动与验证

```bash
## mkdir -p /etc/systemd/system/docker.service.d
usermod -aG docker leryn
systemctl daemon-reload
systemctl restart docker
```
