Ansible 基于 ssh 和 python

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
```bash
kubectl create secret generic ansible-config -n ansible \
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
  name: ansible-ingress
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
  name: ansible-service
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
  name: ansible-deploy
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
                name: ansible-config
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
