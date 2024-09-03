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

```yaml
esxcli system hostname set --fqdn=anna-esxi8-home-ecs-pc.leryn.io
```

## GPU 设备直通
以 PCIe 设备方式直通：

+ （如果设备不支持直通需要通过这种方式开启，但是实操下来有的零件直接开启直通也没有问题）首先需要调整 ESXi 的直通设备映射，

```bash
vi /etc/vmware/passthru.map
```

在末尾添加行：

```plain
<vendor_id> <device_id> default false
```

可以用 `lspci` 获得显卡的 vendor_id 和 device_id，也可以直接在管理台上查看设备详情中的 vendor_id 和 device_id：

```bash
lspci -n
```

修改完成后，需要重新引导宿主机。重新引导后需要在设备界面开启直通。

+ 虚拟机引导方式改为 BIOS
+ 修改虚拟机兼容性，设置到最新版本：例如，ubuntu-server-cloudimg 默认兼容性是 ESXi 5.5，直接修改为支持的最高版本
+ 虚拟机设置内存需要：锁定内存，为虚拟机预留全部内存
+ 虚拟机以 PCIe 设备方式添加 GPU 设备
+ 虚拟机设置高级参数：

```properties
hypervisor.cpuid.v0=FALSE
pciPassthru.use64bitMMIO=TRUE
pciPassthru.64bitMMIOSizeGB=64
```

`pciPassthru.64bitMMIOSizeGB` 的大小应当填为的显存大小下一档 2 的幂次的两倍。

例如 6G 显存显卡，下一档 2 的幂次是 8G，两倍是 16G。

## Cloud-init
参考文档：

+ [VMware Knowledge Base](https://kb.vmware.com/s/article/82250)

前置条件：

+ ESXi 至少需要 7.0 U3 版本及以上
+ VMware Tools 至少需要 11.3.0 版本及以上
+ cloud-init 至少需要 21.1 版本及以上
+ 保证 OVF（cloud-init < 23.1）或 VMware（cloud-init >= 23.1）在 Datasource 列表中

操作步骤：

+ 创建 Metadata 和 Userdata 字段（JSON 或 YAML），然后在虚拟机的高级设置中添加参数：分别表示编码方式是 base64 和 base64 编码后的 JSON/YAML 内容。

:::danger
注意  #cloud-config 必须是 YAML 的第一行

:::

```properties
guestinfo.metadata.encoding=base64
guestinfo.metadata=<base64 metadata.yml>
guestinfo.userdata.encoding=base64
guestinfo.userdata=<base64 userdata.yml>
guestinfo.vendordata.encoding=base64
guestinfo.vendordata=<base64 userdata.yml>
```

也可以使用 gzip+base64 编码

```properties
guestinfo.userdata.encoding=gzip+base64
guestinfo.userdata=$(gzip -c9 <userdata.yaml | { base64 -w0 2>/dev/null || base64; })
```

例如：

```yaml
#cloud-config
users:
  - default
  - name: root
    groups: root
    lock_passwd: false
    plain_text_passwd: P@s5W0rd
```

## Windows 11
### 跳过 TPM 检测
ESXi 平台上安装 Windows 11 提示无法满足系统要求，很可能是因为缺少 TPM 模块。有两种方式：修改注册表跳过 TPM 检测、创建 vTPM 模块（需要 vCenter）。

安装 Windows 11 时，引导界面显示选择语言项时，按住 `Shift + F10` 呼出命令行界面，输入以下命令配置注册表。

```bash
REG ADD HKLM\SYSTEM\Setup\LabConfig /v BypassTPMCheck /t REG_DWORD /d 1
```

