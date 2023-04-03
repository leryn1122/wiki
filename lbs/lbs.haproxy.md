<a name="gOvo8"></a>
# HAProxy + KeepAlived 安装配置
<a name="wOtUW"></a>
## 包管理器
<a name="Bt8QB"></a>
### 环境准备
HAProxy 的环境准备:

- 至少需要两台服务器
- 一个额外可分配的空闲 IP
<a name="AnflW"></a>
### 安装 HAProxy
用包管理器下载安装 HAProxy：
```bash
sudo apt update
sudo apt install haproxy -y
```
修改 HAProxy 的配置文件:
```bash
vim /etc/haproxy/haproxy.cfg
```
```
frontend k8s-master主机名
    bind 0.0.0.0:6443
    mode tcp
    option tcplog
    timeout client 30000
    default_backend k8s-master主机名
backend k8s-master主机名
    mode tcp
    option tcplog
    option tcp-check
    timeout server 30000
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
        server apiserver1 k8s-master-IP地址:6443 check
          ...

frontend k8s-worker主机名-http
    bind 0.0.0.0:80
    mode tcp
    option tcplog
    timeout client 30000
    default_backend k8s-worker主机名-http
backend k8s-worker主机名-http
    mode tcp
    option tcplog
    option tcp-check
    timeout server 30000
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
        server worker1 k8s-master-IP地址:80 check
          ...

frontend k8s-worker主机名-https
    bind 0.0.0.0:443
    mode tcp
    option tcplog
    timeout client 30000
    default_backend k8s-worker主机名-https
backend k8s-worker主机名-https
    mode tcp
    option tcplog
    option tcp-check
    timeout server 30000
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
        server worker1 k8s-master-IP地址:443 check
```
配置说明如下：

| 配置 |  |
| --- | --- |
| global | 全局配置参数段, 主要用来控制 HAProxy 启动前的进程及系统相关设置 |
| defaults | 配置一些默认参数, 如果 frontend,backend,listen 等段未设置则使用 defaults 段设置 |
| listen | 监听配置 |
| frontend | 用来匹配接收客户所请求的域名, URI 等, 并针对不同的匹配, 做不同的请求处理 |
| backend | 定义后端服务器集群, 以及对后端服务器的一些权重、队列、连接数等选项的设置 |

```bash
# 检查配置文件语法
haproxy -c -f /etc/haproxy/haproxy.cfg
# 启动调试功能, 将显示所有连接和处理信息在屏幕
haproxy -d -f /etc/haproxy/haproxy.cfg
# 显示haproxy编译和启动信息
haproxy -vv
```
设置开机启动并启动服务：
```bash
systemctl enable haproxy
systemctl start  haproxy
systemctl status haproxy
```
<a name="Rvbax"></a>
### 安装 KeepAlived
用包管理器下载安装 KeepAlived：
```bash
sudo apt update
sudo apt install keepalived
```
修改 KeepAlived 的配置文件，根据实际的网卡名修改 `interface` 的名称：
```bash
vim /etc/keepalived/keepalived.conf
```
```
! Configuration File for keepalived
vrrp_instance VI_1 {
    state BACKUP
    interface ens6
    virtual_router_id 61
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # notify_master /etc/keepalived/notify_master_mysql.sh
    virtual_ipaddress {
        漂移IP地址/24
    }
}
```
```bash
# 显示KeepAlived版本
keepalived -v
```
设置开机启动并启动服务。
```bash
systemctl enable keepalived
systemctl start  keepalived
systemctl status keepalived
```
<a name="Xljvm"></a>
## 高可用测试
HAProxy 和 KeepAlived 都启动后, `ip a`查看 IP 地址，某节点会占用漂移地址，且`ping`IP 地址可以`ping`通。
```bash
ip a | grep inet
ping <漂移地址>
```
其中一台 HAProxy 关机或禁用网卡模拟断电, 另一台机器将占用虚拟 IP 实现高可用。将断电机器重新启动后，关闭另一台机器，其他 HAProxy 节点将占用虚拟 IP（两节点的话虚拟 IP 漂移到原主机上）。
```bash
# HAProxy节点1
reboot
# HAProxy节点2
ip a | grep inet
ping <漂移地址>

# HAProxy节点2
reboot
# HAProxy节点1
ip a | grep inet
ping <漂移地址>
```
<a name="yYo3b"></a>
## 注意点
之前发生了一个悲惨的故事，发现一台 KeepAlived 服务器上的 DNS 配置有问题，于是重启了网卡，发现 KeepAlived 默默的挂了，重启 KeepAlived 服务即可：
```bash
# 发现DNS配置有问题
cat /etc/resolv.conf
# 重启网卡
service network restart
# 重启KeepAlived
systemctl restart keepalived
```
<a name="E4NCQ"></a>
## Docker 安装
由于官方不提供任何默认配置，所以必须自己编写配置文件 `haproxy.cfg`，没有则无法启动，可以通过如下命令检查：
```bash
docker run -it --rm \
  --name haproxy-syntax-check \
  haproxy \
  haproxy -c -f /usr/local/etc/haproxy/haproxy.cfg
```
```bash
docker run \
  --detach=true \
  --sysctl net.ipv4.ip_unprivileged_port_start=0 \
  --volume=/conf/haproxy:/usr/local/etc/haproxy:ro \
  --name=haproxy \
  --hostname=haproxy \
  haproxy
```
