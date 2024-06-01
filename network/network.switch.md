---
id: network.switch
tags:
- network
- switch
title: "\u4EA4\u6362\u673A"

---


# 交换机


## 简介
交换机（Switch），交换机家族包括了以太网交换机、电话语音交换机、光纤交换机等等。本文都指以太网交换机。
交换机的前身是集线器。集线器工作在物理层（OSI 协议第一层），只支持将单一网线扩展成多网线，可以用于构建子网。它有几个致命的缺点：

- 共享带宽：集线器上的一个网口占用高带宽，其他网口网速都会变慢
- 广播数据：在冲突域内所有网口都会收到广播数据
- 非双工传输模式

交换机是一种数据链路层设备（OSI 协议第二层），这种交换机称为二层交换机，通过 MAC 地址来识别网络中的设备。

- 独立带宽：每个端口都能享受独立的带宽，不会收到其他端口的
- 只能隔离冲突域，不能分割广播域，仍然会引起广播风暴
- 全双工以太网传输模式

此后，又提出了工作在网络层（OSI 协议第三层）的三层交换机，它可以替代或部分替代传统路由器的功能，因此也可以叫路由交换机，同时也能具有接近二层的交换速度。
二层交换机和三层交换机的主要区别在于它们所支持的网络层级和功能。

- 二层交换机只能进行数据链路层的转发，不能进行路由转发和高级网络管理和控制，适用于局域网内的设备通信。
- 三层交换机不仅可以进行数据链路层的转发，还可以进行网络层的路由转发和高级网络管理和控制，适用于不同子网之间的通信和大型企业网络的管理。

根据场景分类可以分成：

- 核心层交换机
- 汇聚层交换机
- 接入层交换机

也可以根据交换速率分类成：

- 百兆交换机
- 千兆交换机
- 万兆交换机

也可以根据接口类型分类成：

- 电口交换机
- 光口交换机
- 电口光口混合交换机


## 交换机堆叠
通过交换机虚拟化技术，既可以在逻辑上集成多台物理连接的交换机，实现拓宽虚拟交换机带宽、提升转发效率的目的，也可以在逻辑上将一台物理交换机虚拟为多台虚拟交换机，实现业务隔离、提升可靠性的目的。
堆叠、M-LAG是目前广泛应用的两种横向虚拟化技术，通过将多台交换设备虚拟为一台设备，共同承担数据转发任务，提升了网络的可靠性。
堆叠（iStack）将多台交换机通过堆叠线缆连接在一起，使多台设备在逻辑上变成一台交换设备，作为一个整体参与数据转发。
作用：

- 扩展端口数量：当接入的用户数增加到原交换机端口密度不能满足接入需求时，可以通过增加新的交换机并组成堆叠而得到满足。
- 扩展带宽：当交换机上行带宽增加时，可以增加新交换机与原交换机组成堆叠系统，将成员交换机的多条物理链路配置成一个聚合组，提高交换机的上行带宽。
- 提高可靠性：堆叠与 Eth-Trunk 一同使用，当堆叠系统中一台设备的上行链路故障，通过该设备的流量可经过堆叠链路进行转发。


## 交换机模拟器

- [大学四年走来，这些网络工程师必备的模拟器我都给你整理好了](https://zhuanlan.zhihu.com/p/117563080)


### Cisco Packet Tracer
完全基于内存的实现

- 支持


### HCL
依赖 VirtualBox


### ENSP
华为现在只向供应商提供 ENSP 安装包，只能通过网络找到分享安装包资源。
依赖 VirtualBox


## 交换机指令
| 
 | **Cisco** | **H3C** | **HW** |
| --- | --- | --- | --- |
| 取消、关闭当前设置 | no | undo | undo |
| 显示查看 | show | display | display |
| 显示当前系统版本 | show version   | display version | display version |
| 设置主机名 | hostname | sysname | sysname |
| 退回上级 | exit | quit | quit |
| 创建用户 | username | local-user | local-user |
| 配置明文密码 | enable password | set authentication  password simple | set authentication  password simple |
| 登出 | exit | logout | logout |
| 重启 | reload | reload | reload |
| 保存当前配置 | write | save | save |
| 删除文件 | delete | delete | delete |
| 查看已保存过的配置 | show startup-config | display saved-configuration | display saved-configuration |
| 显示当前配置 | show running-config | display current-configuration | display current-configuration |
| 删除配置 | erase startup-config | reset saved-configuration | reset saved-configuration |
| 取消所有 debug 命令 | no debug all | ctrl+d | ctrl+d |
| 指定信息中心配置信息 | logging | info-center | info-center |
| 全局模式/系统视图 | en, config terminal  | system-view | system-view |
| 进入线路配置/用户接口视图 | line | user-interface | user-interface |
| 退回用户视图 | end | return | return |
| 启动配置 | start-config | saved-configuration  | saved-configuration  |
| 当前配置 | running-config | current-configuration | current-configuration |
| 禁止、关闭端口 | shutdown | shutdown | shutdown |
| host 名字和 ip 地址对应 | host | ip host | ip host |
| 进入接口 | interface type/number  | interface type/number  | interface type/number  |
| 进入vlan配置vlan管理地址 | interface vlan 1 | interface vlan 1 | interface vlan 1 |
| 定义多个端口的组 | interface rang | interface ethID to ID | interface ethID to ID |
| 设置特权口令 | enable secret | super password | super password |
| 配置接口状态 | duplex (half&#124;full&#124;auto) | duplex (half&#124;full&#124;auto) | duplex (half&#124;full&#124;auto) |
| 配置端口速率 | speed (10/100/1000) | speed (10/100/1000) | speed (10/100/1000) |
| 配置trunk | switchport mode trunk | port link-type trunk | port link-type trunk |
| 添加、删除vlan | vlan ID /no vlan ID  | vlan batch ID /undo vlan batch ID | vlan batch ID /undo vlan batch ID |
| 将端口接入vlan | switchport access  vlan | port acces vlan ID | port default vlan ID |
| 查看接口 | show interface | display interface | display interface |
| 查看vlan | show vlan ID | display vlan ID | display vlan ID |
| 封装协议 | encapsulation | link-protocol | link-protocol |
| 链路聚合 | channel-group 1 mode on | port link-aggregation group 1 | port link-aggregation group 1 |
| 开启三层交换的路由功能 | ip routing | <默认开启> | <默认开启> |
| 开启接口三层功能 | no switchport | <不支持> | <不支持> |
| 对跨以太网交换机的VLAN进行动态注册和删除 | vtp domain | GVRP | GVRP |
| stp配置根网桥 | spanning-tree vlan ID root primary | stp instance id root primary | stp instance id root primary |
| 配置网桥优先级 | spanning-tree vlan ID priority | stp primary value | stp primary value |
| 查看STP配置 | show spanning-tree | dis stp brief | dis stp brief |
| 配置默认路由 | ip route 0.0.0.0  0.0.0.0 | ip route-static  0.0.0.0 0.0.0.0 | ip route-static  0.0.0.0 0.0.0.0 |
| 配置静态路由 | ip route 目标网段 + 掩码 下一跳 | ip route-static  目标网段 + 掩码 下一跳 | ip route-static  目标网段 + 掩码 下一跳 |
| 查看路由表 | show ip route | display ip routing-table | display ip routing-table |
| 启用 RIP、并宣告网段 | router rip /network 网段 | rip /network 网段 | rip /network 网段 |
| 启用ospf | router ospf | ospf | ospf |
| 配置OSPF区域 | network ip 反码 area <area-id> | area <area-id> | area <area-id> |
| 配置RIPV2水平分割 | no auto-summary | rip split-horizon | rip split-horizon |
| 查看路由协议 | show ip protocol | display ip protocol | display ip protocol |
| 扩展访问控制列表 | access-list 100-199 permit/deny protocol source IP + 反码 destination IP + 反码 operator operan

 | rule {normal&#124;special}{permit&#124;deny}{tcp&#124; udp}source {<ip wild>&#124;any}destination  <ip wild>&#124;any}[operate] | rule {normal&#124;special}{permit&#124;deny}{tcp&#124; udp}source {<ip wild>&#124;any}destination  <ip wild>&#124;any}[operate] |
| 配置HSRP组 | standby group-number ip virtual-ip | vrrp vrid number virtual-ip

 | vrrp vrid number virtual-ip

 |
| 配置HSRP优先级 | standby group-number priority | vrrp vrid number  priority | vrrp vrid number  priority |
| 配置HSRP占先权 | standby group-number preempt | vrrp vrid number  preempt-mode  | vrrp vrid number  preempt-mode  |
| 配置端口跟踪 | standby group-number track | <不支持> | <不支持> |
| 配置静态地址转换 | ip nat inside source static | nat server global  <ip> [port] inside <ip> port [protocol] | nat server global  <ip> [port] inside <ip> port [protocol] |



## VLAN


### VLAN
VLAN（Virtual Local Area Network），即虚拟局域网，它是 IEEE 标准化方案。网络管理员可以将物理局域网内的不同用户划分成不同的广播域。每个 VLAN 包含具有享用需求的网络设备，类似于物理的 LAN。但它只是逻辑上的划分，所以并不要求同一 VLAN 限制在同一物理范围中。
VLAN 会在以太帧中添加一个 4 字节的帧标识：

- 前 2 字节是固定的 TPID 标识 0x8100，标识是 IEEE 802.1q 帧标识
- 后 2 字节：
   - 3 位是用户优先级，IEEE 802.1q 不使用
   - 1 位是 CFI 标识：以太网中通常是 0
   - 12 位是 VLAN ID：标识帧所属于的 VLAN，有 0-4095 个 VLAN，其中 VLAN 0 和 VLAN 4095 是保留的。
```
                         +----+----+------+---------------------+
        Ethernet Frame   | DA | SA | Type |  Data               |
                         +----+----+------+---------------------+
                                  / \
                                 /   \
                      +----+----+-----+------+-----------------------+
Tagged Ethernet Frame | DA | SA | Tag | Type | Data                  |
    (IEEE 802.1q)     +----+----+-----+------+-----------------------+
                        ________/     \_________________
                       /                                \
                       +------+----------+-----+---------+
                       | TPID | Priority | CFI | VLAN ID |
                       +------+----------+-----+---------+
                              |                          |
                              |<<------   TCI   ------->>|
                       
```
对于 Access 口和 Trunk 口，接受到数据包会有不同的处理

- Access 一般是服务器和交换机互联的口
- Trunk 一般是交换机和交换机互联的口

PVID = Ported VLAN ID，即端口上的 VLAN ID，一个端口可以属于多个 VLAN，但它只拥有一个 PVID。它会给接收到不带 Tag 头的数据包打上 PVID 作为标识。

| **类型** | **数据** | **操作** |
| --- | --- | --- |
| Access | 收到不含 VLAN 的数据包 | 给数据打上 Access 的 PVID |
| Access | 收到含 VLAN 的数据包 | 如果该 VID 与 Access 的 PVID 相同，则接收，否则丢弃 |
| Access | 发送含 VLAN 的数据包 | 如果该 VID 与 Access 的 PVID 相同，则剥离 VLAN 标签后发送，否则丢弃 |
| Trunk | 收到不含 VLAN 的数据包 | 给数据打上 Trunk 的 PVID |
| Trunk | 收到含 VLAN 的数据包 | 如果该 VID 与 Trunk 的 PVID 相同，则接收，否则丢弃 |
| Trunk | 发送含 VLAN 的数据包 | 如果 Trunk 端口配置允许该 VLAN 的数据包通过，则保留 VID 通过，否则丢弃 |


假设接入交换机 g1/0/24 口接到了上联交换机的 g1/0/12 口，接入交换机的 g1/0/1 接入了一台服务器：
```
                                  1   3   5   7   9  11  13  15  17  19  21  23
                                +---+---+---+---+---+---+---+---+---+---+---+---+
                        SW01    |[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|
                                +---+---+---+---+---+---+---+---+---+---+---+---+
                                |[ ]|[ ]|[ ]|[ ]|[ ]|[x]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]| 
                                +---+---+---+---+---+---+---+---+---+---+---+---+
                                  2   4   6   8  10   |  14  16  18  20  22  24
  SW02                                                |
                                                      |
  1   3   5   7   9  11  13  15  17  19  21  23       |
+---+---+---+---+---+---+---+---+---+---+---+---+     |
|[x]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|     |
+---+---+---+---+---+---+---+---+---+---+---+---+     |
|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[ ]|[x]| ----+
+---+---+---+---+---+---+---+---+---+---+---+---+
  2   4   6   8  10  12  14  16  18  20  22  24
  |
  +--------+
           |
     +-----------+
     |  Server   |
     +-----------+
```
如果服务器想和 SW01 上的相同 VLAN 通信，两者都设置相同的 VLAN Access 口，两者在同一 VLAN 的冲突域内。
如果想和不同 VLAN 通信，那么它的上联需要设置为 Trunk，放行所有需要的 VLAN。
