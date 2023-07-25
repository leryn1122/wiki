<a name="eGF8q"></a>
# MySQL 监控
参考文献：

- [https://github.com/prometheus/mysqld_exporter](https://github.com/prometheus/mysqld_exporter)

1. 创建一个专用于 exporter 的 MySQL 用户：
```bash
CREATE USER 'exporter'@'127.0.0.1' IDENTIFIED BY '*******' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'127.0.0.1';
```

2. 上传 MySQL Exporter 的可执行文件：
```bash
mv mysqld_exporter /usr/bin/
chmod +x /usr/bin/mysqld_exporter
```

3. 创建配置文件：
```bash
mkdir -p /etc/mysqld_exporter
vim /etc/mysqld_exporter/.my.cnf
```
```bash
[client]
host=127.0.0.1
port=3306
user=exporter
password=********
```

4. 创建 systemd 文件：
```bash
vim /usr/lib/systemd/system/mysqld_exporter.service
```
```bash
[Unit]
Description=Mysqld Exporter
Wants=network-online.target
After=network.target
After=network-online.target
Documentation=https://prometheus.io/


[Service]
User=root
ExecStart=/usr/bin/mysqld_exporter \
          --collect.global_status       \
          --collect.global_variables    \
          --collect.info_schema.clientstats     \
          --collect.info_schema.innodb_metrics  \
          --collect.info_schema.innodb_tablespaces      \
          --collect.info_schema.innodb_cmp      \
          --collect.info_schema.innodb_cmpmem   \
          --collect.info_schema.processlist     \
          --collect.info_schema.query_response_time     \
          --collect.info_schema.replica_host    \
          --collect.info_schema.tables          \
          --collect.info_schema.tablestats      \
          --collect.info_schema.schemastats     \
          --collect.info_schema.userstats       \
          --config.my-cnf=/etc/mysqld_exporter/.my.cnf
[Install]
WantedBy=multi-user.target
```
```bash
systemctl restart mysqld_exporter.service
systemctl enable mysqld_exporter.service
systemctl status mysqld_exporter.service
```

5. 修改 Prometheus 配置
```yaml
  - job_name: "external/mysql_exporter"
    scrape_interval: 30s
    static_configs:
      - targets:
          - xxx.xxx.xxx.xxx:9104
    relabel_configs:
      - target_label: cluster
        replacement: XXXXXXX
```
