---
id: infra.ventoy
tags:
- infra
- ventoy
title: Ventoy

---


# Ventoy
参考文档：

- [Ventoy](https://www.ventoy.net/cn/)
- [GitHub - ventoy/Ventoy: A new bootable USB solution.](https://github.com/ventoy/Ventoy)

Ventoy 是一个制作可启动 U 盘的开源工具。有了 Ventoy 之后，安装系统无需反复格式化 U 盘。


## 安装手册
:::danger
注意首次安装 Ventoy 会格式化 U 盘，该 U 盘内的所有数据都会抹除且无法恢复。
:::
首次安装 Ventoy 需要格式化 U 盘，请先备份数据。后续升级 Ventoy 不会格式化数据，但是注意区分安装和升级。下载操作系统对应的 Ventoy 安装包：

- [https://github.com/ventoy/Ventoy/releases/download/v1.0.97/ventoy-1.0.97-linux.tar.gz](https://github.com/ventoy/Ventoy/releases/download/v1.0.97/ventoy-1.0.97-linux.tar.gz)
- [https://github.com/ventoy/Ventoy/releases/download/v1.0.97/ventoy-1.0.97-livecd.iso](https://github.com/ventoy/Ventoy/releases/download/v1.0.97/ventoy-1.0.97-livecd.iso)
- [https://github.com/ventoy/Ventoy/releases/download/v1.0.97/ventoy-1.0.97-windows.zip](https://github.com/ventoy/Ventoy/releases/download/v1.0.97/ventoy-1.0.97-windows.zip)

安装完后可以看到 U 盘上产生了新的分区。
![Ventoy 引导页面](https://cdn.nlark.com/yuque/0/2024/png/26002940/1713631946034-b7e45d07-df3e-4ba8-a41b-03beaab3ebf0.png#averageHue=%23dcddda&clientId=uafa2fbea-31cb-4&from=ui&id=ud110ad0c&originHeight=480&originWidth=640&originalType=binary&ratio=2&rotation=0&showTitle=true&size=17198&status=done&style=none&taskId=u6a50d07c-a276-4420-8585-a6abb567eb1&title=Ventoy%20%E5%BC%95%E5%AF%BC%E9%A1%B5%E9%9D%A2 "Ventoy 引导页面")
