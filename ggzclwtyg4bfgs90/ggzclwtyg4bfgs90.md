如何在 kubernetes 中 tcpdump 容器的网络包：

- kubectl debug
- tcpdump 容器网卡


```bash
# crictl ps 定位到容器
crictl ps | grep xxx

# 
crictl inspect xxxx
crictl inspect 8bdb34c7dc620 | \
  jq '.info.runtimeSpec.linux.namespaces[] | select(.type == "network").path'
```
