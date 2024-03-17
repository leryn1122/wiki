---
id: cri.network
tags:
- cri
- docker
- network
title: "Docker \u7F51\u7EDC"

---


# Docker 网络
在安装 docker 时，会自动在宿主机上创建三个网络，用 `docker network ls` 可以进行查看：
```bash
$ docker network ls

NETWORK ID     NAME      DRIVER    SCOPE
647d344c40c0   bridge    bridge    local
e7a2dda75feb   host      host      local
1885060cbe3f   none      null      local
```


## none 网络
none 网络就是没有网络，挂在 none 网络下面的容器只有 lo 网卡。封闭意味着隔离，一些对安全性要求高并且不需要联网的应用可以使用 none 网络。
```bash
$ docker run -it --rm --network=none busybox:1.30.1
/ # ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```


## host 网络
连接到 host 网络的容器共享宿主机的网络栈，容器的网络配置跟宿主机的完全相同。容器中能看到宿主机的所有网卡，连 hostname 也跟宿主机一样。
直接使用宿主机网络栈，最大的好处就是性能，如果容器对网络传输效率有较高要求，则可以选择 host 网络。当然不便之处就是牺牲一些灵活性，比如要考虑端口冲突问题。
Docker host 的另一个用途是让容器可以直接配置 host 网路。比如某些跨 host 的网络解决方案，其本身也是以容器方式运行的，这些方案需要对网络进行配置，比如管理 iptables。
```bash
$ docker run -it --rm --network=host busybox:1.30.1

/ # hostname
tencent-01
```


## bridge 网络

