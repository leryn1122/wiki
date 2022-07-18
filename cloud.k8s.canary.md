

参考文档:

- [k8s-deployment-strategies - Github](https://github.com/ContainerSolutions/k8s-deployment-strategies)
- [NGINX Ingress Controller Annotations for Canary - NGINX Ingress Controller 官网](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#canary)
- [ingress实现灰度发布 - CSDN](https://blog.csdn.net/zyl290760647/article/details/104640652/)
- [kubernetes基于nginx-ingress进行蓝绿部署、金丝雀发布（canary）- CSDN](https://blog.csdn.net/weixin_46754666/article/details/124832391?)

- `nginx.ingress.kubernetes.io/canary-by-header`：基于 Request Header 的流量切分，适用于灰度发布以及 A/B 测试。当 Request Header 设置为 always时，请求将会被一直发送到 Canary 版本；当 Request Header 设置为 never时，请求不会被发送到 Canary 入口；对于任何其他 Header 值，将忽略 Header，并通过优先级将请求与其他金丝雀规则进行优先级的比较。
- `nginx.ingress.kubernetes.io/canary-by-header-value`：要匹配的 Request Header 的值，用于通知 Ingress 将请求路由到 Canary Ingress 中指定的服务。当 Request Header 设置为此值时，它将被路由到 Canary 入口。该规则允许用户自定义 Request Header 的值，必须与上一个 annotation (即：canary-by-header）一起使用。
- `nginx.ingress.kubernetes.io/canary-by-header-pattern`：他会让 ``
- `nginx.ingress.kubernetes.io/canary-by-cookie`：基于 Cookie 的流量切分，适用于灰度发布与 A/B 测试。用于通知 Ingress 将请求路由到 Canary Ingress 中指定的服务的cookie。当 cookie 值设置为 always时，它将被路由到 Canary 入口；当 cookie 值设置为 never时，请求不会被发送到 Canary 入口；对于任何其他值，将忽略 cookie 并将请求与其他金丝雀规则进行优先级的比较。
- `nginx.ingress.kubernetes.io/canary-weight`：基于服务权重的流量切分，适用于蓝绿部署，权重范围 0 - 100 按百分比将请求路由到 Canary Ingress 中指定的服务。权重为 0 意味着该金丝雀规则不会向 Canary 入口的服务发送任何请求。权重为 100 意味着所有请求都将被发送到 Canary 入口。

注意：金丝雀规则按优先顺序进行如下排序

canary-by-header - > canary-by-cookie - > canary-weight
