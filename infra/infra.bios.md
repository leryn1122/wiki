---
id: infra.bios
tags:
- bios
- infra
- uefi
title: BIOS & UEFI

---


# BIOS & UEFI
当我们启动电脑时，主板 ROM 内存储的固件（firmware）将会运行：它将负责电脑的加电自检（power-on self test），可用内存（available RAM）的检测，以及 CPU 和其它硬件的预加载。这之后，它将寻找一个可引导的存储介质（bootable disk），并开始引导启动其中的内核（kernel）。
x86 架构支持两种固件标准： BIOS（[Basic Input/Output System](https://en.wikipedia.org/wiki/BIOS)）和 UEFI（[Unified Extensible Firmware Interface](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)）。其中 BIOS 标准显得陈旧而过时，但实现简单，并为 1980 年代后的所有 x86 设备所支持；相反地，UEFI 更现代化，功能也更全面，但开发和构建更复杂（至少从我的角度看是如此）。
计算机启动时，不停按着固定键可以进入 BIOS/UEFI。不同厂商提供的设备，键位不同。
