<a name="ZD5Li"></a>
## Kubectl 通用命令

Kubectl 很多命令和 docker 是通用的, 无非是增加 `-n <namespace>` 这个参数:

```bash
# 进入 pod
kubectl exec -it <pod> -n <namespace> -- bash

# 查看 pod 日志
kubectl log -f <pod> -n <namespace>
```

其他命令也很通俗易懂, 总体而言以下几种命令使用频率比较高:

```bash
# 查看 kubernetes 资源的配置:
kubectl get <resource> <name> -n <namespace> -o yaml

# 修改 kubernetes 资源的配置
kubectl edit <resource> <name> -n <namespace>

# 删除无用的 replica
kubectl delete replicaset $(kubectl get replicaset.apps -A | awk '$3==0{printf "%s -n %s\n",$2,$1}')
```

一个更加复杂的例子, 比如查看 Ingress-nginx 的 `nginx.conf`:

```bash
kubectl exec -it $(kubectl get pods -n gateway  | grep -E ^gateway | awk '{print $1}') \
  -n ingress-nginx -- bash -c "cat /etc/nginx/nginx.conf" | less 
```

![20211119211339.jpg](./../assets/1645168735400-7705f3f6-64a7-4bc3-a9e2-e8068f49d5f8.jpeg)
<a name="gzIoo"></a>
## 
