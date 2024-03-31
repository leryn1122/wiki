---
id: os.smbios
tags:
- dmi
- infra
- smbios
title: SMBIOS

---


# SMBIOS
参考资料：

- [SMBIOS | DMTF](https://www.dmtf.org/standards/smbios)
- [https://www.dmtf.org/sites/default/files/standards/documents/DSP0134_3.7.0.pdf](https://www.dmtf.org/sites/default/files/standards/documents/DSP0134_3.7.0.pdf)
- [https://www.dmtf.org/sites/default/files/standards/documents/DSP0130.pdf](https://www.dmtf.org/sites/default/files/standards/documents/DSP0130.pdf)


## 介绍

- SMBIOS（System Management BIOS）是一个主板或系统制造者以标准格式显示产品管理信息所需遵循的统一规范。
- DMI（Desktop Management Interface）就是帮助收集电脑系统信息的管理系统，DMI 信息的收集必须在严格遵照 SMBIOS 规范的前提下进行。

在 Linux 平台下读取以下三个文件（通常是前两个）可以 s 获得 SMBIOS 数据。

- `/sys/firmware/dmi/tables/smbios_entry_point`（SMBIOS Entrypoint）s
- `/sys/firmware/dmi/tables/DMI`（SMBIOS Table data）
- `/dev/mem`（FreeBSD）

SMBIOS 只能获得板载设备，例如集成显卡、板载网卡。如果插在插槽上的，例如独立显卡、独立网卡，只能获得插槽口上的信息，而不能获得所有详细的设备信息。


## 命令行
使用命令行接口：
```bash
dmidecode -s xxx
# Valid string keywords are:
#  bios-vendor
#  bios-version
#  bios-release-date
#  bios-revision
#  firmware-revision
#  system-manufacturer
#  system-product-name
#  system-version
#  system-serial-number
#  system-uuid
#  system-sku-number
#  system-family
#  baseboard-manufacturer
#  baseboard-product-name
#  baseboard-version
#  baseboard-serial-number
#  baseboard-asset-tag
#  chassis-manufacturer
#  chassis-type
#  chassis-version
#  chassis-serial-number
#  chassis-asset-tag
#  processor-family
#  processor-manufacturer
#  processor-version
#  processor-frequency

dmidecode -t xxx
# Valid type keywords are:
#  bios
#  system
#  baseboard
#  chassis
#  processor
#  memory
#  cache
#  connector
#  slot
```


## API
可以通过第三方库，访问 SMBIOS：

- Rust
   - `smbios-lib`
   - `dmidecode-rs`
- Golang
   - `github.com/siderolabs/go-smbios/smbios`
   - `github.com/digitalocean/go-smbios/smbios`
