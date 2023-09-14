
# Kubernetes 监控

## Kubernetes Event Exporter
参考文档：

- [Kubernetes Events - Kubernetes 官方文档](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/event-v1/)
- [opsgenie/kubernetes-event-exporter - GitHub](https://github.com/opsgenie/kubernetes-event-exporter)
- [resmoio/kubernetes-event-exporter - GitHub](https://github.com/resmoio/kubernetes-event-exporter)

这个插件可以采集 Kubernetes 中的积压的事件，因为 Kubernetes 事件堆积大约 15 分钟后就会被清理，因此要追溯问题非常复杂。<br />它相当于：
```bash
kubectl get events -A
```
参考该仓库的 `README.md` 即可, 有两种安装方式：

- 宿主机部署：通过 `~/.kube/config` ；
- Kubernetes 内部署；

我们直接使用 Prometheus 对应的 `serviceaccount`，对应的 `clusterrole` 的 RBAC 权限如下：
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: event-exporter
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: event-exporter
  namespace: monitoring
rules:
  - apiGroups:
      - ""
    resources:
      - nodes
      - services
      - endpoints
      - pods
      - nodes/proxy
      - events
      - replicasets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - configmaps
      - nodes/metrics
    verbs:
      - get
  - nonResourceURLs:
      - /metrics
    verbs:
      - get
  - apiGroups:
      - apps
    resources:
      - services
      - replicasets
      - deployments
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - policy
    resources:
      - poddisruptionbudgets
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: event-exporter
  namespace: monitoring
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: event-exporter
subjects:
  - kind: ServiceAccount
    name: event-exporter
    namespace: monitoring
```
我比较推荐使用该方库下的 YAML 直接在集群内部署. 因为我的集群里已经拥有了 Prometheus 以及他对应的 RBAC。所以删除 `00-roles.yaml`，并且修改 `02-deployment.yaml`里的 `serviceAccoutName` ,直接运行以下脚本：
```bash
#!/usr/bin/env bash

find *.yaml | xargs -L1 kubectl apply -n monitoring -f
```
如果需要使用 EFK 来采集日志（其他的 Receiver 可以参考 GitHub），那么以下这个配置可以一共参考, 有几点说明：

- ElasticSearch 使用的认证有请必须用引号括起来，否则生成的 Http 认证请求头可能无法通过认证；
- 多个集群采集到统一索引下时，不妨使用 layouts 添加一个表示的集群字段；
- 由于 exporter 和 ElasticSearch 的吞吐量不一致可能会阻塞一部分事件的发送，这目前还是在 [Issue-159](https://github.com/opsgenie/kubernetes-event-exporter/issues/159) 和 [Issue-192](https://github.com/opsgenie/kubernetes-event-exporter/issues/192) 中；
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: event-exporter-cfg
  namespace: monitoring
data:
  config.yaml: |-
    logLevel: error
    logFormat: json
    throttlePeriod: 300
    kubeQPS: 300
    kubeBurst: 300
    route:
      routes:
        - match:
            - receiver: "es"
            - receiver: "stdout"
    receivers:
      - name: "es"
        elasticsearch:
          hosts:
            - http://xxx.xxx.xxx.xxx:9200
          username: "elastic"
          password: "xxxxxxx"
          index: kube-events
          indexFormat: "kube-events-{2006.01.02}"
          #layouts:
          #  cluster: "my-k8s-cluster"
      - name: "stdout"
        stdout: { }
        
```
然后在 Kibana 上，可以设置一个 Index Pattern 为 `kube-events*` 即可，访问 ElasticSearch 查看索引：
```bash
curl -XPOST http://xxx.xxx.xxx.xxx:9200/_xpack/sql?format=json \
     -H "Authorization: Basic xxxxxxx" \
     -H "Content-Type: application/json" \
     -d '{"query": "select firstTimestamp, metadata.namespace, reason, message from \"kube-event*\" limit 10"}'
```
