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
helm repo add minio https://charts.min.io/
helm repo update

helm install minio minio/minio -n oss \
  --set consoleIngress.enabled=true                    \
  --set consoleIngress.hosts={oss-console.leryn.top}   \
  --set consoleIngress.ingressClassName=nginx          \
  --set ingress.enabled=true                           \
  --set ingress.hosts={oss.leryn.top}                  \
  --set ingress.ingressClassName=nginx                 \
  --set mode=standalone                                \
  --set networkPolicy.allowExternal=true               \
  --set persistence.existingClaim=oss-data-pvc         \
  --set persistence.size=5Gi                           \
  --set replicas=1                                     \
  --set resources.requests.memory=128Mi                \
  --set rootPassword=xxxxxxxxxx                        \
  --set rootUser=admin                                 \
  --set securityContext.fsGroup=0                      \
  --set securityContext.runAsUser=0                    \
  --set securityContext.runAsGroup=0
```