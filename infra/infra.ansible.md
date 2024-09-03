---
id: infra.ansible
tags:
- ansible
- cli
- infra
title: "Ansible - \u81EA\u52A8\u5316\u8FD0\u7EF4"

---
# Ansible - 自动化运维
Ansible 在我的工作内容中有两个用途：

+ 批量操作服务器：例如一个非常真实的场景是网络安全组要求为某个网段的服务器加装安全探针、升级内核等等，一个 Ansible 脚本可以完成数百台服务器的安装；
+ 监控系统时更加接近 Shell 命令的内容，例如 ping 我们会用 REST API 调用 Ansible Semaphore 来完成。

## Ansible
Ansible 基于 ssh 和 Python

Ansible 很重要的特性是所有操作的都具备幂等性。

```bash
ansible -i hosts -m ping

ansible-playbook -i hosts main.yaml
```

## Semaphore - Ansible 控制台
参考文档：

+ [Introduction - Semaphore Docs](https://docs.ansible-semaphore.com/)
+ [Ansible Semaphore](https://demo.ansible-semaphore.com/)
+ [API - Ansible Semaphore](https://www.semui.co/api-docs/)
+ [Build software better, together](https://github.com/fiftin/ansible-semaphore-deploy-demo)

基于 Golang 开发的 Ansible 控制台，对外暴露 REST API，搭配数据库持久化操作记录。

以下用 Kubernetes 部署一个 Ansible Semaphore：

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
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ansible
  labels:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
spec:
  rules:
    - host: ansible.mydomain.com
      http:
        paths:
          - service:
              name: ansible
              port:
                number: 80
            path: /
            pathType: ImplementationSpecific
---
apiVersion: v1
kind: Service
metadata:
  name: ansible
  labels:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 3000
  selector:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ansible
  labels:
    app.kubernetes.io/instance: ansible
    app.kubernetes.io/name: ansible
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

          ports:
            - containerPort: 3000
              name: http
              protocol: TCP
          livenessProbe:
            tcpSocket:
              port: http
          readinessProbe:
            tcpSocket:
              port: http
      restartPolicy: Always
```

## 常用 API
域名使用 `ansible.mydomain.com`，由于是集群内部可以直接用 Kubernetes Service 的名字 `ansible.ansible`。

```bash
# 登录并保存 cookie
curl -ik -XPOST \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{"auth": "admin", "password": "cangetin"}' \
  -c /tmp/semaphore-cookie \
  http://ansible.mydomain.com/api/auth/login

# 带 cookie 生成永久 token
curl -ik -XPOST \
  -b /tmp/semaphore-cookie \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  http://ansible.mydomain.com/api/user/tokens
# id 就是 token
# {"id":"rqxvmlutifi-yfxphq2k0vf3t-xxxxxxxxxxxxx","created":"2022-09-06T08:10:11Z","expired":false,"user_id":1}

# 之后的请求 Basic Auth 带上 Token 即可
# 查看 Project 下的 Task
curl -ik -XGET \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer rqxvmlutifi-yfxphq2k0vf3t-xxxxxxxxxxxxx' \
  http://ansible.mydomain.com/api/projects

# 启动一个 Task
curl -ik -XPOST \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer rqxvmlutifi-yfxphq2k0vf3t-xxxxxxxxxxxxx' \
  -d '{"template_id": 3}' \
  http://ansible.mydomain.com/api/project/1/tasks
```

