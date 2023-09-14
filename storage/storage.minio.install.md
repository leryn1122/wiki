
# MinIO 安装手册
参考文档：

- [https://docs.min.io/](https://docs.min.io/)
- [https://docs.minio.org.cn/docs/](https://docs.minio.org.cn/docs/)

## Docker 安装
这里：

- 端口 9000 因为被其他应用占用了，所以改用 9002
- 密码要至少 8 位，否则会报错。
```bash
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
比较新的 minio 镜像中以下两个环境变量已经废弃，请在管理端手动创建新用户。
```bash
  --env="MINIO_ACCESS_KEY=admin" \
  --env="MINIO_SECRET_KEY=admin" \
```

## Helm 安装
参考文档：

- [https://github.com/minio/minio/tree/master/helm/minio](https://github.com/minio/minio/tree/master/helm/minio)

较新的 MinIO 调整了网关，暂时使用老版本的 Chart。
```bash
helm repo add minio https://charts.min.io/
helm repo update

helm install minio minio/minio -n oss \
  --version 4.0.14 \
  --set consoleIngress.enabled=true                    \
  --set consoleIngress.hosts={oss-console.leryn.top}   \
  --set consoleIngress.ingressClassName=nginx          \
  --set ingress.enabled=true                           \
  --set ingress.hosts={oss.leryn.top}                  \
  --set ingress.ingressClassName=nginx                 \
  --set-string ingress.annotations."nginx.ingress.kubernetes.io/proxy-body-size"=0 \
  --set-string consoleIngress.annotations."nginx.ingress.kubernetes.io/proxy-body-size"=0 \
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
