
![](https://s3.leryn.top/website/image/minio.png#crop=0&crop=0&crop=1&crop=1&height=61&id=MQwuM&originHeight=226&originWidth=1500&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=407)
<a name="mKUb8"></a>
# MinIO
参考文档:

- [MinIO Quickstart Guide - 官网](https://docs.min.io/)
- [MinIO 文档 - 官网](http://docs.minio.org.cn/docs/)
- [S3 - markdown](docs_aws_aws-s3-overview)
- [MinIO 管理端](https://minio-ui.leryn.top)
<a name="Q3xXj"></a>
## Docker 安装

这里:

- 端口 9000 因为被其他应用占用了, 所以改用 9002
- 密码要至少 8 位, 否则会报错.

```
docker run \
  --detach=true \
  --env="MINIO_ROOT_USER=admin" \
  --env="MINIO_ROOT_PASSWORD=admin" \
  --publish=9000:9000 \
  --publish=9001:9001 \
  --restart=always \
  --volume=/data/minio:/data \
  --volume=/conf/minio:/root/.minio \
  --name=minio \
  --hostname=minio \
  minio/minio:latest server /data --console-address ":9001"
```

比较新的 minio 镜像中以下两个环境变量已经废弃, 请在管理端手动创建新用户.

```
  --env="MINIO_ACCESS_KEY=admin" \
  --env="MINIO_SECRET_KEY=admin" \
```


```bash
helm install aws-s3 bitnami/minio -n aws-s3 \
  --set auth.rootUser="admin" \
  --set auth.rootPassword="admin" \
  --set ingress.enabled="true" \
  --set ingress.hostname="oss-adm.leryn.top" \
  --set apiIngress.enabled="true" \
  --set apiIngress.hostname="oss.leryn.top" \
  --set persistence.enabled="false"
  
```

