
```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: acme-issuer
spec:
  acme:
    email: user@example.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: example-issuer-account-key
    solvers:
    - dns01:
        webhook:
          groupName: $WEBHOOK_GROUP_NAME
          solverName: $WEBHOOK_SOLVER_NAME
          config:
            ...
            <webhook-specific-configuration>
```
```bash
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  # 每个域名不要重复
  name: example-crt
  # 证书生成的工作空间，不指定也可以
  namespace: test-dns
spec:
  #生成后证书的配置文件名称
  secretName: example-crt
  issuerRef:
    name: dnspod
    kind: ClusterIssuer
    group: cert-manager.io
  dnsNames:
  # 填写需要证书的域名: 可以是泛域名
  - xxx.xxxx.cn
```
