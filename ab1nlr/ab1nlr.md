

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-driver-s3-secret
  namespace: kube-system
stringData:
  # endpoint: 如果是 AWS S3 需要设置为 https://s3.<region>.amazonaws.com
  # region:   如果不在 AWS S3 上, 请 region 将其设置为 ""
  accessKeyID: "t3yw+5CmRP"
  secretAccessKey: "kVpAwMUT3nxV&B!NOjRp"
  endpoint: https://minio.lishuai.fun
  region: ""
```


