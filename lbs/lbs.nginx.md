
# Nginx
![](./../assets/1706973752886-351af435-9e51-49bc-b5e6-10cdfe5ada32.png)

## 安装手册
参考手册：

- [nginx: download](http://nginx.org/en/download.html)
- [GitHub - nginx/nginx: An official read-only mirror of http://hg.nginx.org/nginx/ which is updated hourly. Pull requests on GitHub cannot be accepted and will be automatically closed. The proper way to submit changes to nginx is via the nginx development mailing list, see http://nginx.org/en/docs/contributing_changes.html](https://github.com/nginx/nginx)
- [Nginx 安装配置 | 菜鸟教程](https://www.runoob.com/linux/nginx-install-setup.html)
- [nginx安装及其配置详细教程 - 博客园](https://www.cnblogs.com/lywJ/p/10710361.html)

Nginx 强烈不推荐使用源码编译的方式安装，除非需要额外的模块。

### 包管理器
```bash
sudo apt update -y
sudo apt install -y nginx

sudo systemctl restart nginx
sudo systemctl status  nginx
sudo systemctl enable  nginx
```

### Docker 安装
```bash
docker run \
  --detach=true \
  --publish=80:80 \
  --publish=443:443 \
  --volume=/etc/nginx/ssl:/etc/nginx/ssl:ro \
  --volume=/etc/nginx/sites-enabled:/etc/nginx/sites-enabled:ro \
  nginxinc/nginx-unprivileged:1.24-bullseye
```

### Nginx 动态模块
参考文档：

- [How to Compile Dynamic Modules for NGINX and NGINX Plus](https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/)
```bash
# 下载对应版本的源码包
nginx -v

NGINX_VERSION=0.18.0

wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
tar -xzvf nginx-*.tar.gz

git clone https://github.com/perusio/nginx-hello-world-module.git
```

## Nginx 配置
参考文档：

- [NGINXConfig | DigitalOcean](https://www.digitalocean.com/community/tools/nginx?global.app.lang=zhCN)

推荐以上网站可以在线模块化生成 nginx 配置文件。
