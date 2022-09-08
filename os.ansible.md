<a name="JPshA"></a>
# Ansible
![ansible.svg](./assets/1662607034984-1ed9ae65-5eaa-4edb-8ea6-fe5b61bb9f79.svg)"<br />Ansible 在我的工作内容中有两个用途:

- 批量操作服务器: 例如一个非常真实的场景是网络安全组要求为某个网段的服务器加装安全探针、升级内核等等, 一个 Ansible 脚本可以完成数百台服务器的安装;
- 监控系统时更加接近 Shell 命令的内容, 例如 ping 我们会用 REST API 调用 Ansible Semaphore 来完成.
<a name="McX3m"></a>
## Ansible
Ansible 基于 ssh 和 Python<br />Ansible 很重要的特性是所有操作的都具备幂等性.

```bash
ansible -i hosts -m ping

ansible-playbook -i hosts main.yaml
```

<a name="LsvN0"></a>
## Semaphore - Ansible 控制台
参考文档:

- 
- 
- 
- 

基于 Golang 开发的 Ansible 控制台, 对外暴露 REST API, 搭配数据库持久化操作记录.<br />以下用 Kubernetes 部署一个 Ansible Semaphore:
```bash
kubectl create secret generic ansible-secret -n ansible \
  --from-literal=SEMAPHORE_DB_USER=semaphore \
  --from-literal=SEMAPHORE_DB_PASS=******** \
  --from-literal=SEMAPHORE_DB_HOST=******** \
  --from-literal=SEMAPHORE_DB_PORT=3306 \
  --from-literal=SEMAPHORE_DB=semaphore \
  --from-literal=SEMAPHORE_PLAYBOOK_PATH=/tmp/semaphore/ \
  --from-literal=SEMAPHORE_ADMIN_PASSWORD=cangetin \
  --from-literal=SEMAPHORE_ADMIN_NAME=admin \
  --from-literal=SEMAPHORE_ADMIN_EMAIL=admin@localhost \
  --from-literal=SEMAPHORE_ADMIN=admin \
  --from-literal=SEMAPHORE_ACCESS_KEY_ENCRYPTION=gs72mPntFATGJs9qK0pQ0rKtfidlexiMjYCH9gWKhTU=
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  labels:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
  name: ansible
spec:
  rules:
    - host: ansible.mydomain.com
      http:
        paths:
          - backend:
              serviceName: ansible-service
              servicePort: 80
            path: /
            pathType: ImplementationSpecific
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
  name: ansible
spec:
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 3000
  selector:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
  name: ansible
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/instance: ansible
      app.kubernetes.io/name: ansible
  template:
    metadata:
      labels:
        app.kubernetes.io/instance: ansible
        app.kubernetes.io/name: ansible
    spec:
      containers:
        -
        	name: ansible
          env:
            - name: TZ
              value: Asia/Shanghai
          envFrom:
            - secretRef:
                name: ansible-secret
          image: ansiblesemaphore/semaphore:latest
          imagePullPolicy: Always
          livenessProbe:
            tcpSocket:
              port: http
          ports:
            - containerPort: 3000
              name: http
              protocol: TCP
          readinessProbe:
            tcpSocket:
              port: http
      restartPolicy: Always
```
