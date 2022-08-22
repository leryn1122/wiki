<a name="flm16"></a>
# Redis Exporter Prometheus 监控<br />![](https://cloud.drone.io/api/badges/oliver006/redis_exporter/status.svg#crop=0&crop=0&crop=1&crop=1&id=eBwiq&originHeight=20&originWidth=91&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![](https://coveralls.io/repos/github/oliver006/redis_exporter/badge.svg?branch=master#crop=0&crop=0&crop=1&crop=1&id=d7dDD&originHeight=20&originWidth=99&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![](https://codecov.io/gh/oliver006/redis_exporter/branch/master/graph/badge.svg#crop=0&crop=0&crop=1&crop=1&id=EFE0h&originHeight=20&originWidth=112&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![](https://img.shields.io/docker/pulls/oliver006/redis_exporter.svg#crop=0&crop=0&crop=1&crop=1&id=zUskV&originHeight=20&originWidth=118&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
参考文档:

- [redis_exporter - GitHub 主页](https://github.com/oliver006/redis_exporter)
- [Redis Grafana Dashboard - Grafana 官网](https://grafana.com/grafana/dashboards/11835-redis-dashboard-for-prometheus-redis-exporter-helm-stable-redis-ha)

这个 Exporter 已经被 Prometheus 官方采纳了, 并作为官网解决方案了<br />新建配置文件: `/etc/redis_exporter/redis_exporter.conf`
```toml
OPTIONS="-redis.addr=xxx.xxx.xxx.xxx:6379/26379 -redis.password='PASSWORD'"
```
创建 `/lib/systemd/system/redis-exporter.service` :
```toml
[Unit]
Description=Redis Exporter
Documentation=https://github.com/oliver006/redis_exporter
Wants=network-online.target
After=syslog.target
After=redis.service
# After=redis-sentinel.service

[Service]
Type=simple
EnvironmentFile=-/etc/redis_exporter/redis_exporter.conf
ExecStart=/opt/redis_exporter/redis_exporter $OPTIONS

[Install]
WantedBy=multi-user.target
```
启动服务并设置开机自启动:
```bash
systemctl start  redis-exporter.service
systemctl enable redis-exporter.service
```
启动后可以访问端口获得指标:
```bash
curl -XGET http://localhost:9121/metrics

# 实际采集走这个接口, 可以采到集群下不同机器的性能状况
curl -XGET http://localhost:9121/scrape?target=xxx.xxx.xxx.xxx
```
修改 Prometheus 配置文件:
```yaml
  - job_name: 'REDIS_CULSTER'
    static_configs:
      - targets:
        - redis://xxx.xxx.xxx.xxx:6379
        - ...
        - redis://xxx.xxx.xxx.xxx:26379
        - ...
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: xxx.xxx.xxx.xxx:9121
  - job_name: 'redis_exporter'
    static_configs:
      - targets:
        - xxx.xxx.xxx.xxx:9121
```
Grafana 导入对应的仪表盘即可:
```
11835
```
其中监控仪表盘上的 `Memory Usage` 可能因为没有设置 Redis 的最大内存而错误的显示 `∞%`.
```bash
#!/usr/bin/env bash

set -eux

cd -P "$(dirname "${0-$BASHSOURCE}")" || exit 1

mkdir -p /opt/redis_exporter
mkdir -p /etc/redis_exporter

cp -a ./redis_exporter         /opt/redis_exporter
cp -a ./redis_exporter.conf    /etc/redis_exporter/redis_exporter.conf
cp -a ./redis-exporter.service /lib/systemd/system/redis-exporter.service

systemctl start  redis-exporter
systemctl enable redis-exporter

curl -XGET http://localhost:9121/metrics
```
