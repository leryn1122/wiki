<a name="x2QMA"></a>
# Ingress Controller

Ingress 就是**入口**的意思, 那么 Ingress Controller 就是 入口控制器了. 它为了解决 Kubernetes 集群上同一入口的负载均衡. Ingress Controller 有不同的实现方式, 云原生的 traefik 或者是传统的 nginx. 这里我个人更加倾向于 Ingress-nginx, 因为我个人更加熟悉 nginx 配置文件.<br />![20220217001612.png](./../assets/1648298721391-71350fe3-57e3-47ed-a828-f774a636d15e.png)
<a name="s30le"></a>
## Nginx Ingress Controller 安装步骤

参考文档:

- [NGINX Ingress Controller - Github](https://github.com/bitnami/charts/tree/master/bitnami/nginx-ingress-controller)
- [NGINX Ingress Controller - Values.yaml - Github](https://github.com/bitnami/charts/blob/master/bitnami/nginx-ingress-controller/values.yaml)
- [NGINX Ingress Controller ConfigMaps - NGINX Ingress Controller 官网](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/)

使用 Helm 安装 Ingress Controller, 注意安装后的提示信息.

```bash
helm search repo nginx-ingress

kubectl create ns ingress-nginx

# 默认安装
helm install gateway bitnami/nginx-ingress-controller -n ingress-nginx

# 如果需要改动默认配置需要下载并改动values.yaml
helm pull bitnami/nginx-ingress-controller
tar -xf *.gz
helm install ingress-nginx nginx-ingress-controller -n ingress-nginx
```

安装后会打印一段信息, 提示如何安装示例:

```yaml
Release "ingress-nginx" has been upgraded. Happy Helming!
NAME: ingress-nginx
LAST DEPLOYED: Mon Feb 14 16:11:08 2022
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
CHART NAME: nginx-ingress-controller
CHART VERSION: 9.1.5
APP VERSION: 1.1.1

** Please be patient while the chart is being deployed **

The nginx-ingress controller has been installed.

Get the application URL by running these commands:

 NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        You can watch its status by running 'kubectl get --namespace ingress-nginx svc -w ingress-nginx-nginx-ingress-controller'

    export SERVICE_IP=$(kubectl get svc --namespace ingress-nginx ingress-nginx-nginx-ingress-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "Visit http://${SERVICE_IP} to access your application via HTTP."
    echo "Visit https://${SERVICE_IP} to access your application via HTTPS."

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: ingress-nginx
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                service:
                  name: example-service
                  port:
                    number: 80
              path: /
              pathType: Prefix
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: ingress-nginx
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

配置一个自己的 Ingress 和 Service 就可以测试了.<br />调试的时候可以通过如下命令来查看 Ingress-nginx 的配置, 如果服务发现配置成功, 可以在其中发现对应服务的配置.

```bash
kubectl get pods -n ingress-nginx
kubectl exec -it ingress-nginx-nginx-ingress-controller-57b6f9794f-zkzrj -n gateway \
  -- bash -c "cat /etc/nginx/nginx.conf"
```

<a name="bjIt3"></a>
## 配置 Https 证书

需要配置 Https 证书. 获得一下两个值, 我这里是用 acme.sh 生成的自授权证书:

**方法1**

```bash
kubectl create secret tls leryn.top \
  --cert '/root/.acme.sh/*.leryn.top/fullchain.cer'   \
  --key  '/root/.acme.sh/*.leryn.top/*.leryn.top.key'
```


**方法2**

```bash
# tls.crt
cat '/root/.acme.sh/*.leryn.top/fullchain.cer'   | base64 -w 0
# tls.key
cat '/root/.acme.sh/*.leryn.top/*.leryn.top.key' | base64 -w 0
```

将上面的 `tls.crt`和 `tls.key`填入一下 yaml:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: leryn.top
data:
  tls.crt: 
  tls.key: 
type: kubernetes.io/tls
```

```bash
kubectl apply -f leryn.top.yaml
```

更新 Ingress-controller:

```bash
helm install gateway bitnami/nginx-ingress-controller -n gateway \
  --set extraArgs.default-ssl-certificate="default/leryn.top" \
  --set config.use-gzip="true" \
  --set config.gzip-level="6" \
  --set config.gzip-min-length="1k" \
  --set defaultBackend.enabled="false"
```
