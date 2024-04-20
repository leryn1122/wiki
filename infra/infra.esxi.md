---
id: infra.esxi
tags:
- esxi
- infra
- virt
- vm
title: ESXi & vSphere

---


# ESXi & vSphere
vSphere 是 VMWare 整个套件的集合，它包括了 ESXi，vCenter 等等。
```bash
wget http://cloud-images.ubuntu.com/releases/22.04/release/ubuntu-22.04-server-cloudimg-amd64.img

virt-customize -a ubuntu-22.04-server-cloudimg-amd64.img --root-password password:ubuntu
```


## GPU 设备直通
以 PCIe 设备方式直通：

- （如果设备不支持直通需要通过这种方式开启，但是实操下来有的零件直接开启直通也没有问题）首先需要调整 ESXi 的直通设备映射，
```bash
vi /etc/vmware/passthru.map
```
在末尾添加行：
```
<vendor_id> <device_id> default false
```
可以用 `lspci` 获得显卡的 vendor_id 和 device_id，也可以直接在管理台上查看设备详情中的 vendor_id 和 device_id：
```bash
lspci -n
```
修改完成后，需要重新引导宿主机。重新引导后需要在设备界面开启直通。

- 虚拟机引导方式改为 BIOS
- 虚拟机设置内存需要：锁定内存，为虚拟机预留全部内存
- 虚拟机以 PCIe 设备方式添加 GPU 设备
- 虚拟机设置高级参数：
```properties
hypervisor.cpuid.v0=FALSE
pciPassthru.use64bitMMIO=TRUE
pciPassthru.64bitMMIOSizeGB=64
```
`pciPassthru.64bitMMIOSizeGB` 的大小应当填为的显存大小下一档 2 的幂次的两倍。例如 6G 显存显卡，下一档 2 的幂次是 8G，两倍是 16G。
