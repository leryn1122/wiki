---
id: zn802ge2xw2ykzmp
tags: []
title: CNI

---
### Multus
参考文档：

+ [https://github.com/k8snetworkplumbingwg/multus-cni/blob/master/deployments/multus-daemonset-thick.yml](https://github.com/k8snetworkplumbingwg/multus-cni/blob/master/deployments/multus-daemonset-thick.yml)

Multus 支持挂载多个网卡到 Kubernetes 的 Pod 中。使用 NetworkAttachmentDefinition CRD 来配置 CNI 的配置文件。

```yaml
annotations:
  k8s.v1.cni.cncf.io/networks: default/macvlan-conf@eth1, default/macvlan-conf@eth2
```

### Whereabouts
参考文档：

+ [https://github.com/k8snetworkplumbingwg/whereabouts/tree/master/deployment/whereabouts-chart](https://github.com/k8snetworkplumbingwg/whereabouts/tree/master/deployment/whereabouts-chart)

whereabouts 插件可以在集群级别地址池中动态分配唯一 IP ，与 `host-local` 插件的区别是后者只能在本地节点上动态分配 IP。

### OVS CNI
参考文档：

+ [https://github.com/k8snetworkplumbingwg/ovs-cni](https://github.com/k8snetworkplumbingwg/ovs-cni)

OVS CNI 可以在宿主机的 Open vSwitch 上创建 VLAN ID 等等。在使用这个技术前，需要现在宿主机上安装 Open vSwitch，创建对应的虚拟网桥，这里假设叫 `vlan-br`。

```bash
apt install -y openvswitch-switch

ovs-vsctl add vlan-br
```

### NetworkAttachmentDefinition
现在需求是需要创建

+ eth0 默认网卡
+ eth1 需要使用 macvlan 技术，集群内互通，使用 192.168.100.0/22
+ vlan 需要为每个 Pod 指定单独的 VLAN，固定 IP 地址和 MAC 地址

解决方案：

+ 多网卡肯定要使用 multus 插件
+ eth0 用 CNI 挂载的默认网卡
+ eth1 用 macvlan 插件 + whereabouts 分配 IP 池
+ vlan 网卡用 ovs 插件 + static 分配固定 IP + tuning 修改 MAC 地址，tuning 修改 MAC 地址需要在 podSecurityContext 中添加 `NET_ADMIN`。
+ 最后在 Pod 上打上注解：

```yaml
annotations:
  k8s.v1.cni.cncf.io/networks: default/macvlan-conf@eth1, default/ovs-vlan1000-conf@vlan
```

```yaml
---
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: macvlan-conf
spec:
  config: |-
    {
      "cniVersion": "0.4.0",
      "plugins": [
        {
          "type": "macvlan",
          "master": "eth1",
          "mode": "bridge",
          "capabilities": {
            "ips": true
          },
          "ipam": {
            "type": "whereabouts",
            "range": "192.168.103.1-192.168.103.254/22",
            "range_start": "192.168.103.1",
            "range_end": "192.168.103.254",
            "routes": [
              {
                "dst": "192.168.100.0/22"
              }
            ],
            "gateway": "192.168.100.1",
            "exclude": [
               "192.168.2.229/30",
               "192.168.2.236/32"
            ]
          }
        }
      ]
    }
---
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: ovs-vlan1000-conf
  annotations:
    k8s.v1.cni.cncf.io/resourceName: ovs-cni.network.kubevirt.io/br1
spec:
  config: |-
    {
      "cniVersion": "0.4.0",
      "plugins": [
        {
          "type": "ovs",
          "bridge": "vlan-br",
          "vlan": "1000",
          "interface_type": "system",
          "ipam": {
            "type": "static",
            "addresses": [
              {
                "address": "192.168.18.1/16"
              }
            ]
          }
        },
        {
          "type": "tuning",
          "capabilities": {
            "mac": true
          },
          "mac": "00:11:22:33:44:55"
        }
      ]
    }

```

