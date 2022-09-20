<a name="HLLWK"></a>
# Kubernetes的三种外部访问方式：NodePort、LoadBalancer 和 Ingress
【编者的话】本文分析了 NodePort，LoadBalancer 和 Ingress 这三种访问服务方式的使用方式和使用场景，指出了各自的优缺点，帮助用户基于自己的场景做出更好的决策。

最近有些同学问我 NodePort，LoadBalancer 和 Ingress 之间的区别。它们都是将集群外部流量导入到集群内的方式，只是实现方式不同。让我们看一下它们分别是如何工作的，以及你该如何选择它们。**如果你想和更多Kubernetes技术专家交流，可以加我微信liyingjiese，备注『加群』。群里每周都有全球各大公司的最佳实践以及行业最新动态**。

**注意**：这里说的每一点都基于[Google Kubernetes Engine](https://cloud.google.com/gke)。如果你用 minikube 或其它工具，以预置型模式（om prem）运行在其它云上，对应的操作可能有点区别。我不会太深入技术细节，如果你有兴趣了解更多，[官方文档](https://kubernetes.io/docs/concepts/services-networking/service/)是一个非常棒的资源。<br />ClusterIP 服务是 Kubernetes 的默认服务。它给你一个集群内的服务，集群内的其它应用都可以访问该服务。集群外部无法访问它。

ClusterIP 服务的 YAML 文件类似如下：

<a name="wbCtY"></a>
### ClusterIP
apiVersion: v1 kind: Service metadata:   name: my-internal-service selector:     app: my-app spec: type: ClusterIP ports:   - name: http port: 80 targetPort: 80 protocol: TCP 

如果 从Internet 没法访问 ClusterIP 服务，那么我们为什么要讨论它呢？那是因为我们可以通过 Kubernetes 的 proxy 模式来访问该服务！

启动 Kubernetes proxy 模式：

这样你可以通过Kubernetes API，使用如下模式来访问这个服务：

要访问我们上面定义的服务，你可以使用如下地址：

有一些场景下，你得使用 Kubernetes 的 proxy 模式来访问你的服务：

这种方式要求我们运行 kubectl 作为一个未认证的用户，因此我们不能用这种方式把服务暴露到 internet 或者在生产环境使用。<br />NodePort 服务是引导外部流量到你的服务的最原始方式。NodePort，正如这个名字所示，在所有节点（虚拟机）上开放一个特定端口，任何发送到该端口的流量都被转发到对应服务。

NodePort 服务的 YAML 文件类似如下：

NodePort 服务主要有两点区别于普通的“ClusterIP”服务。第一，它的类型是“NodePort”。有一个额外的端口，称为 nodePort，它指定节点上开放的端口值 。如果你不指定这个端口，系统将选择一个随机端口。大多数时候我们应该让 Kubernetes 来选择端口，因为如评论中 thockin 所说，用户自己来选择可用端口代价太大。<br />这种方法有许多缺点：

基于以上原因，我不建议在生产环境上用这种方式暴露服务。如果你运行的服务不要求一直可用，或者对成本比较敏感，你可以使用这种方法。这样的应用的最佳例子是 demo 应用，或者某些临时应用。<br />LoadBalancer 服务是暴露服务到 internet 的标准方式。在 GKE 上，这种方式会启动一个 [Network Load Balancer](https://cloud.google.com/compute/docs/load-balancing/network/)，它将给你一个单独的 IP 地址，转发所有流量到你的服务。

如果你想要直接暴露服务，这就是默认方式。所有通往你指定的端口的流量都会被转发到对应的服务。它没有过滤条件，没有路由等。这意味着你几乎可以发送任何种类的流量到该服务，像 HTTP，TCP，UDP，Websocket，gRPC 或其它任意种类。

这个方式的最大缺点是每一个用 LoadBalancer 暴露的服务都会有它自己的 IP 地址，每个用到的 LoadBalancer 都需要付费，这将是非常昂贵的。<br />有别于以上所有例子，Ingress 事实上不是一种服务类型。相反，它处于多个服务的前端，扮演着“智能路由”或者集群入口的角色。

你可以用 Ingress 来做许多不同的事情，各种不同类型的 Ingress 控制器也有不同的能力。

GKE 上的默认 ingress 控制器是启动一个 [HTTP(S) Load Balancer](https://cloud.google.com/compute/docs/load-balancing/http/)。它允许你基于路径或者子域名来路由流量到后端服务。例如，你可以将任何发往域名 foo.yourdomain.com 的流量转到 foo 服务，将路径 yourdomain.com/bar/path 的流量转到 bar 服务。

GKE 上用 [L7 HTTP Load Balancer](https://cloud.google.com/compute/docs/load-balancing/http/) 生成的 Ingress 对象的 YAML 文件类似如下：

[![image.png](./../assets/bc92fdf5f0ae4f6d2024a15d056d68b9.png)<br />$ kubectl proxy --port=8080 <br />http://localhost:8080/api/v1/proxy/namespaces/<NAMESPACE>/services/<SERVICE-NAME>:<PORT-NAME>/ <br />http://localhost:8080/api/v1/proxy/namespaces/default/services/my-internal-service:http/ 
<a name="dr8ci"></a>
#### 何时使用这种方式？

1. 由于某些原因，你需要调试你的服务，或者需要直接通过笔记本电脑去访问它们。
1. 容许内部通信，展示内部仪表盘等。
<a name="tvKmJ"></a>
### NodePort
[![image.png](./../assets/58174ac44fdbacbbc89cec648260fcdf.png)<br />apiVersion: v1 kind: Service metadata:   name: my-nodeport-service selector:     app: my-app spec: type: NodePort ports:   - name: http port: 80 targetPort: 80 nodePort: 30036 protocol: TCP 
<a name="UjpOo"></a>
#### 何时使用这种方式？

1. 每个端口只能是一种服务
1. 端口范围只能是 30000-32767
1. 如果节点/VM 的 IP 地址发生变化，你需要能处理这种情况。
<a name="n8Px0"></a>
### LoadBalancer
[![image.png](./../assets/d8631b315a7acdd6926ec5405ed1043f.png)
<a name="RWGff"></a>
#### 何时使用这种方式？
<a name="q70Ts"></a>
### Ingress
[![image.png](./../assets/ab886a9dd4e912cf6f5a1f3ed983ac4c.png)<br />apiVersion: extensions/v1beta1 kind: Ingress metadata: name: my-ingress spec: backend: serviceName: other servicePort: 8080 rules: - host: foo.mydomain.com http:   paths:   - backend:       serviceName: foo       servicePort: 8080 - host: mydomain.com http:   paths:   - path: /bar/*     backend:       serviceName: bar       servicePort: 8080
