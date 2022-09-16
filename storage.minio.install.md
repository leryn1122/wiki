<a name="mKUb8"></a>
# MinIO 安装手册
参考文档:

- [https://docs.min.io/](https://docs.min.io/)
- [http://docs.minio.org.cn/docs/](http://docs.minio.org.cn/docs/)
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
<a name="OjoQs"></a>
## Helm 安装
参考文档:

- [https://github.com/minio/minio/tree/master/helm/minio](https://github.com/minio/minio/tree/master/helm/minio)
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
```properties
consoleIngress.enabled=true
consoleIngress.hosts[0]=oss-console.leryn.top
ingress.enabled=true
ingress.hosts[0]=oss.dev.leryn.top
mode=standalone
networkPolicy.allowExternal=true
persistence.existingClaim=minio-pvc
persistence.size=5Gi
replicas=1
resources.requests.memory=128Mi
rootPassword=
rootUser=admin
securityContext.fsGroup=0
securityContext.runAsUser=0
securityContext.runAsGroup=0
```
