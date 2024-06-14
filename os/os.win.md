---
id: os.win
tags:
- windows
title: Windows

---


# Windows 封装
```bash
# 激活 Windows
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms kms.03k.org
slmgr /ato

# 启用管理员
net user administrator /active: yes

# 禁用 OOBE：覆盖 OOBE 二进制
XCOPY C:\windows\System32\svchost.exe C:\windows\System32\oobe\audit.exe /X
```
安装常见软件：

- OpenSSH Server：[Releases · PowerShell/Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/releases)
- Dism++：[Releases · Chuyu-Team/Dism-Multi-language](https://github.com/Chuyu-Team/Dism-Multi-language/releases)
- 


