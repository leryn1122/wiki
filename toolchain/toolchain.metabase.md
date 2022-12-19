<a name="eqT9l"></a>
# Metabase 安装手册
参考文档：

- [https://www.metabase.com/](https://www.metabase.com/)
- [https://www.metabase.com/docs/latest/installation-and-operation/running-metabase-on-docker](https://www.metabase.com/docs/latest/installation-and-operation/running-metabase-on-docker)
- [https://github.com/metabase/metabase](https://github.com/metabase/metabase)
- [https://github.com/PublicI/metabase-helm](https://github.com/PublicI/metabase-helm)

Metabase 是一款开源但商用的数据看板 Web 界面，有免费试用版本。使用 Clojure 和 Typescript 开发。只能查看数据、制作图表，不能修改数据。支持多数据源管理、多种数据库、元数据采集、权限管理等等。<br />首先创建数据库，用于存储应用自身数据，目前有 H2/Postgrel/MySQL 三种方式，我选用 MySQL。<br />使用如上 Chart 安装即可，如果出现 Kubernetes 版本不适配可以直接下载我提供的版本（来源自阿里云仓库的缓存 v0.3.2，修改到了兼容 Kubernetes v1.18-1.22）：
```bash
helm add repo infra harbor.leryn.top/infra/chartrepo
helm pull infra/metabase
```
```bash
helm install metabase stable/metabase -n metabase \
  --set database.dbname=metabase \
  --set database.host=xxx.xxx.xxx.xxx \
  --set database.password=xxxxxxxxx \
  --set database.port=3306 \
  --set database.type=mysql \
  --set database.username=metabase \
  --set image.repository=harbor.leryn.top/infra/metabase \
  --set image.tag=v0.44.5 \
  --set ingress.enabled=true \
  --set ingress.hosts={metabase.mydomain.com} \
  --set timeZone=Asia/Shanghai
```

