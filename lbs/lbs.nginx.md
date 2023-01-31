![](http://nginx.org/nginx.png#id=XbNaU&originHeight=72&originWidth=352&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
<a name="p3D5W"></a>
# Nginx 安装手册

参考手册:

- [Nginx 下载 - 官网](http://nginx.org/en/download.html)
- [Nginx 安装配置 - Runoob](https://www.runoob.com/linux/nginx-install-setup.html)
- [Nginx 安装及其配置详细教程 - cnblogs - 作者: lywJee](https://www.cnblogs.com/lywJ/p/10710361.html)
- [Nginx 安装 - oschina - 作者: staybug](https://my.oschina.net/staybug/blog/4254456?hmsr=kaifa_aladdin)

Nginx 强烈不推荐使用源码编译的方式安装
<a name="P9CZi"></a>
## 包管理器
<a name="XVdu0"></a>
### 安装步骤

```bash
sudo apt update -y
sudo apt install -y nginx
```

<a name="pjk1Q"></a>
### 启动与验证

```bash
sudo systemctl restart nginx
```
<a name="dYlzA"></a>
## Docker 安装

```bash
docker run \
  --detach=true \
  --publish=80:80 \
  --publish=443:443 \
  --volume=/etc/nginx/ssl:/etc/nginx/ssl:ro \
  --volume=/etc/nginx/sites-enabled:/etc/nginx/sites-enabled:ro \
  --name=nginx \
  --hostname=nginx \
  nginx:latest
```


<a name="GUgc2"></a>
# Nginx

参考文档:

- [nginx 编译动态模块 - 官网](https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/)
- <br />

```bash
# 下载对应版本的源码包
nginx -v

NGINX_VERSION=0.18.0

wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
tar -xzvf nginx-*.tar.gz

git clone https://github.com/perusio/nginx-hello-world-module.git

```
