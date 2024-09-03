---
id: network.dns
tags: []
title: DNS

---
# DNS
## DNS 配置
### 公网备案
国内公网使用域名需要工信部备案，否则无法在公网开通 80/443 端口的服务（~~内网可无视~~）。客服小姐姐也会在域名备案时提供人工服务。域名备案后，必须在主页的最下方展示域名备案，并跳转到工信部官网备案查询。

### 内网 DNS
一般公司内网都会有一个内网 DNS。有的甚至是，集团母公司一个，本公司一个。这种情况下，申请一个域名需要通过繁复的审批流程，短则一周，长则一到两个月，其中可能还会有网络安全扫描等。为了工作方便，所以建议在平台内部部署一个小的内网 DNS，作为公司 DNS 的子节点，维护一些平台内的域名会比较方便。

有些大公司甚至使用每台主机都有独立的主机名、域名及解析。这种场景在内网 DNS 的支持下，所有配置 IP 地址的方式，都将替换为域名。

### HTTP TLS 证书
阿里云可以免费领取 20 个域名证书，仅支持单域名，不支持泛域名。

需要泛域名 HTTP TLS 证书推荐使用`Let's Encryption`生成自授权证书。具体见 [HTTPS 自授权证书](https://www.yuque.com/leryn/wiki/https)。

### DNS配置
+ `*`表示泛域名；
+ `@`表示空域名；
+ `_acme-challenge`用于`acme.sh`域名认证；

| 主机 | 记录类型 | 值 |
| --- | --- | --- |
| * | CNAME | leryn.top |
| www | CNAME | leryn.top |
| @ | A | IP 地址 |
| _acme-challenge | TXT | 授权码 |


### hosts 文件
这里由于我是单服务器，没有集群，所以可以使用本地 DNS 配置 `/etc/hosts`，将域名指向 localhost。流量会先走本地 DNS，并且是环回地址，不会从外部网卡出网，可以大幅缩短网络 IO 时间。

**<font style="color:#E8323C;">请不要</font>**在公司中使用任何形式的配置代理和域名解析，这会导致服务器的配置混乱，无法管理。因为事实上每个环节都可以配置代理和域名解析：从 `/etc/hosts`、服务器 `HTTP_PROXY` 环境变量、systemd、containerd 配置文件、容器内、CoreDNS 等等，一旦出现了域名解析出现问题将花费大量时间排查。

```bash
sudo vim /etc/hosts
```

```plain
127.0.0.1 leryn.top
127.0.0.1 *.leryn.top
```

Linux 和 Windows 对应的 hosts 文件地址：

```plain
/etc/hosts
C:\Windows\System32\drivers\etc\hosts
```

## CoreDNS
搭建内网 CoreDNS 服务器，用 etcd 作为后端存储。

### etcd
启动 etcd：

```plain
vim /etc/systemd/system/etcd.service

systemctl enable  etcd.service
systemctl restart etcd.service
systemctl status  etcd.service
```

```toml
[Unit]
Description=Etcd Server
Documentation=https://github.com/etcd-io/etcd
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
Environment="ETCD_IMAGE_TAG=v3.5.13"
Environment="ETCD_DATA_DIR=/var/lib/etcd"
#Environment="ETCD_SSL_DIR=/etc/ssl/certs"
Environment="ETCD_OPTS=--name coredns \
  --listen-client-urls https://localhost:2379 \
  --advertise-client-urls https://localhost:2379 \
  --listen-peer-urls https://localhost:2380 \
  --initial-advertise-peer-urls https://localhost:2380 \
#  --initial-cluster coredns01=https://localhost:2380,coredns02=https://localhost:2380,coredns03=https://localhost:2380 \
#  --initial-cluster-token mytoken \
#  --initial-cluster-state new \
#  --client-cert-auth \
#  --trusted-ca-file /etc/ssl/certs/etcd-root-ca.pem \
#  --cert-file /etc/ssl/certs/s1.pem \
#  --key-file /etc/ssl/certs/s1-key.pem \
#  --peer-client-cert-auth \
#  --peer-trusted-ca-file /etc/ssl/certs/etcd-root-ca.pem \
#  --peer-cert-file /etc/ssl/certs/s1.pem \
#  --peer-key-file /etc/ssl/certs/s1-key.pem \
  --auto-compaction-retention 1"
ExecStart=/usr/local/bin/etcd
ExecStop=/bin/kill -HUP $MAINPID
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

### coredns
配置 CoreDNS 配置文件：

```plain
mkdir -p /etc/coredns
vim /etc/coredns/Corefile
```

```plain
. {
    forward . 8.8.8.8:53
    cache
}

leryn.io:53 {
    etcd {
        stubzones
        path /intranet
        endpoint http://localhost:2379
        upstream 8.8.8.8:53 8.8.4.4:53 /etc/resolv.conf
        fallthrough
    }
    cache
    loadbalance
    log
}
```

启动 CoreDNS：

```plain
vim /etc/systemd/system/coredns.service


systemctl stop systemd-resolved.service
systemctl disable systemd-resolved.service

systemctl enable  coredns.service
systemctl restart coredns.service
systemctl status  coredns.service
```

```toml
[Unit]
Description=Coredns
Documentation=https://github.com/coredns/coredns
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/coredns -conf /etc/coredns/Corefile
ExecStop=/bin/kill -HUP $MAINPID
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

添加一条 DNS 记录：

```plain
etcdctl put /intranet/io/leryn/coredns-ubt22-ecs-pc '{"host":"192.168.1.6"}'
```

