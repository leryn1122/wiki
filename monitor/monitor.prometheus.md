
# Prometheus 入门
Prometheus Server 负责去每个 Exporter 上拉去数据到内存里，默认是`/metrics`。
AlertManager 根据数据计算规则，发送报警通知。
TSDB：可以用 InfluxDB，或者更新的 VictoriaMetrics。
Grafana 作为前端页面发送 PromQL 查询语句给 TSDB，渲染视图。

![image.png](./../assets/1690264013236-1965f86e-58f2-4035-acc0-ca982588c467.png)

```yaml
server:
  ingress:
    enabled: true
    host:
      - prometheus.mydomain.com
```
`registry.k8s.io/kube-state-metrics/kube-state-metrics` 的镜像可能拉不下来，可以改用 `bitnami/kube-state-metrics`。
