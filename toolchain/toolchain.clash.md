<a name="x0xAr"></a>
# Clash 安装手册
本文只介绍如何将 Clash 部署到服务端，实现上网。
<a name="SINyW"></a>
## Clash 手动安装手册
安装客户端：
```bash
wget https://github.com/Dreamacro/clash/releases/download/v1.10.6/clash-linux-amd64-v1.10.6.gz
gzip -d clash-linux-amd64-v1.10.6.gz
mv clash-linux-amd64-v1.10.6 /usr/local/bin
cd /usr/local/bin
ln -sf clash-linux-amd64-v1.10.6 clash
```
创建配置文件的文件夹：
```bash
mkdir -p /etc/clash
```
把 PC 客户端的配置文件拷贝到这里，取名 `config.yml`。并创建 Clash 代理的配置文件。
```bash
cat <<EOF> /etc/clash/clash.env
https_proxy=http://127.0.0.1:7890
http_proxy=http://127.0.0.1:7890
all_proxy=soks5://127.0.0.1:7891
EOF
```
创建 Systemd 文件，并设置开启自启动。
```bash
touch /etc/systemd/system/clash.service
```
```properties
[Unit]
Description=Clash Proxy
Documentation=https://github.com/Dreamacro/clash
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
Restart=always
RestartSec=10s
EnvironmentFile=/etc/clash/clash.env
ExecStart=/usr/local/bin/clash -d /etc/clash &
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process

[Install]
WantedBy=multi-user.target
```
```bash
systemctl restart clash.service
systemctl enable  clash.service
systemctl status  class.service
```

