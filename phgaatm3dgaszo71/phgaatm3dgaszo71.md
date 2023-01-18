参考文档：

- [一文读懂容器存储接口 CSI - 作者：阿里巴巴开源 - 知乎](https://zhuanlan.zhihu.com/p/364255271)
<a name="yRxdd"></a>
## CSI Sidecar 组件介绍
<a name="nynJL"></a>
### Node Driver Registar
将外部 CSI 插件注册到 Kubelet，Kubelet 绑定 Unix socket 来调用外部 CSI 插件的函数。

<a name="Vszqy"></a>
### External Provisioner

<a name="sETfq"></a>
### External Attacher
External-Attacher 内部会时刻 watch 集群中的 VolumeAttachment 资源和 PersistentVolume

<a name="PFBRt"></a>
## CSI 接口
存储厂商需实现 CSI 插件的三大接口：`IdentityServer`、`ControllerServer`、`NodeServer`。
<a name="zpSYb"></a>
### IdentityServer
IdentityServer 主要用于认证 CSI 插件的身份信息
<a name="n9j1V"></a>
### ControllerServer
ControllerServer 主要负责存储卷及快照的创建/删除以及挂接/摘除操作
<a name="HO7Ym"></a>
### NodeServer
NodeServer 主要负责存储卷挂载/卸载操作


