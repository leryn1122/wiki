<a name="vyPba"></a>
# Kubernetes 运维
<a name="QzB7H"></a>
## Kubernetes 运维常见场景
<a name="w7OfI"></a>
### 集群证书问题

目前 Kubernetes 是由 `kubeadm` 安装的 1.18 版本，默认证书有效期是1年<br />通过以下命令统一更新由 `kubeadm` 管理的所有证书：

```bash
kubeadm alpha certs renew all
```

注意此时kubenetes组件 api-server, scheduler 以及 controller manager 镜像内部的证书未更新 (可能是kubernetes 1.18版本bug)<br />我们需要重新启动镜像:

```bash
docker ps | grep -E 'k8s_kube-apiserver|k8s_kube-controller-manager|k8s_kube-scheduler' | awk -F ' ' '{print $1}' | xargs docker restart
```

如果未重启，此后证书过期，将导致Pod调度等问题。此时通过docker logs命令可观察到上述容器报告以下错误：

> Unable to authenticate the request due to an error: x509: certificate has expired or is not yet valid


<a name="pBVQr"></a>
### 强制删除 namespace

有时候 `kubectl` 删除 namespace 时, namespace 一直处于 **terminating**. 这分两种场景:

- 有时是 namespace 资源比较多, 回收比较慢, 这种情况只需要等待即可
- 有时是卡死了, 可以使用如下脚本删除

```bash
# 登录 k8s-master 节点, 查看 namespace 是否已经是 terminating 的状态了.
kubectl get ns | grep terminating | grep xxxx

# 导出 namespace 的 json 数据并删除 spe c中的所有内容
kubectl get namespace xxxx -o json > xxxx.json

# 新开一个 kubectl 的代理
kubectl proxy --port=8081

# 调用 finalize 的 api
curl -k -H "Content-Type: application/json" \
  -XPUT --data-binary @xxxx.json http://127.0.0.1:8081/api/v1/namespaces/xxxx/finalize
```

<a name="lqEpO"></a>
### 操作 Etcd

```bash
export ETCDCTL_API=3
alias etcdctl='etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key'
```
