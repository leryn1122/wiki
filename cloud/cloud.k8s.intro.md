<a name="bdWAr"></a>
# Kubernetes 介绍
参考文档:

- [https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#service-v1-core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#service-v1-core)

Kubernetes 是 Google 推出的容器编排技术, 由 Golang 开发. Google 背书, 通过自家大量的容器管理和运维, 目前在容器编排技术上只此一家.

传统部署、虚拟机部署、容器化部署三者的区别:<br />![](./../assets/1658306431889-3842a32a-6c54-4190-a244-6459419ec94b.svg)


<a name="JrVDW"></a>
# Kubernetes 架构
Kubernetes 分为 master 和 worker 两类节点:

- master 负责调度, 需要多台做纯粹的冗余保持 HA
- worker 负责实际负载

master 由三部分组成:

- API Server: 它是一个接口接受所有外界操作 Kubernetes 的指令
- Scheduler: 调度器
- Controller: 控制器, 通过 ControlLoop 时刻维持容器
- Etcd: 基于 Raft 算法的分布式键值数据库


<a name="g60Ba"></a>
### Kubernetes 和 Spring Cloud 的对比

![image.png](./../assets/1646906397611-efb9adb3-5892-405d-8cf6-45eb664ef8d7.png)

