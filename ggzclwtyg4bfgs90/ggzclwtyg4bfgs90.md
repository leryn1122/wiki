
# tcpdump
如何在 kubernetes 中 tcpdump 容器的网络包：

- kubectl debug
- tcpdump pod的网卡

## kubectl debug
需要 kubernetes 更改配置再重启 kubelet
```bash
kubectl debug -it ${pod_name} --image=rstp/net-tools:latest --target=${container_name} -- bash
```

## tcpdump pod的网卡
```bash
# crictl ps 定位到容器
crictl ps | grep xxx

# crictl 查看 netns
crictl inspect 8bdb34c7dc620
crictl inspect 8bdb34c7dc620 | \
  jq '.info.runtimeSpec.linux.namespaces[] | select(.type == "network").path'

# 进入容器的 netns
nsenter --net=/proc/2152524/ns/net
ip a

# pod 内的 eth0 就是 宿主机上的 257 号网卡
256: eth0@if257: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 42:0a:b3:3c:c9:f6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.0.0.51/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::400a:b3ff:fe3c:c9f6/64 scope link 
       valid_lft forever preferred_lft forever

# 回到宿主机的 netns
nsenter --net=/proc/1/ns/net
ip a

# 宿主机的 257 网卡就是 pod 的网卡
257: lxc9749d6440b99@if256: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 42:8b:80:f6:e7:7d brd ff:ff:ff:ff:ff:ff link-netns cni-cc201ee2-49ad-7f8b-2124-4c9685c4a995
    inet6 fe80::408b:80ff:fef6:e77d/64 scope link 
       valid_lft forever preferred_lft forever

tcpdump -i lxc9749d6440b99 -w /tmp/packet_capture.pcap
```
