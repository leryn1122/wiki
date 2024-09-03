---
id: monitor.prometheus.elastic
tags:
- elasticsearch
- prometheus
title: "Elasticsearch Prometheus \u76D1\u63A7"

---
# Elasticsearch Prometheus 监控
参考文档：

+ [https://github.com/prometheus-community/elasticsearch_exporter](https://github.com/prometheus-community/elasticsearch_exporter)

下载二进制包安装：

```bash
vim /usr/lib/systemd/system/elasticsearch_exporter.service
```

```toml
[Unit]
Description=Elasticsearch Exporter
Wants=network-online.target
After=network.target
After=network-online.target
Documentation=https://github.com/prometheus-community/elasticsearch_exporter/

[Service]
User=root
ExecStart=/opt/elasticsearch_exporter/elasticsearch_exporter \
          --es.all              \
          --es.indices          \
          --es.indices_settings \
          --es.shards           \
          --es.snapshots        \
          --es.timeout=10s      \
          --web.listen-address=":9114" \
          --web.telemetry-path="/metrics" \
          --es.uri http://elastic:xxxxx@localhost:9200/

[Install]
WantedBy=multi-user.target
```

```bash
systemctl start  elasticsearch_exporter.service
systemctl enable elasticsearch_exporter.service
systemctl status elasticsearch_exporter.service
```

Prometheus 配置：

```yaml
  - job_name: external/elastichsearch_exporter
    honor_timestamps: true
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: /metrics
    scheme: http
    follow_redirects: true
    enable_http2: true
    relabel_configs:
      - separator: ;
        regex: (.*)
        target_label: cluster
        replacement: my-es-cluster
        action: replace
    static_configs:
      - targets:
          - xxx.xxx.xxx.xxx:9114
        labels:
          env: "dev"
```

导入安装包内的`dashboard.json`到 Grafana 即可。

