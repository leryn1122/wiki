参考文档：

- [一文读懂容器存储接口 CSI - 作者：阿里巴巴开源 - 知乎](https://zhuanlan.zhihu.com/p/364255271)
- [https://kubernetes-csi.github.io/docs/introduction.html](https://kubernetes-csi.github.io/docs/introduction.html)

## CSI Sidecar 组件介绍

### Node Driver Registar
将外部 CSI 插件注册到 Kubelet，Kubelet 绑定 Unix socket 来调用外部 CSI 插件的函数。


### External Provisioner


### External Attacher
External-Attacher 内部会时刻 watch 集群中的 VolumeAttachment 资源和 PersistentVolume


## CSI 接口
存储厂商需实现 CSI 插件的三大接口：`IdentityServer`、`ControllerServer`、`NodeServer`。

### IdentityServer
IdentityServer 主要用于认证 CSI 插件的身份信息

### ControllerServer
ControllerServer 主要负责存储卷及快照的创建/删除以及挂接/摘除操作

### NodeServer
NodeServer 主要负责存储卷挂载/卸载操作


