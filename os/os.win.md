---
id: os.win
tags:
- windows
title: Windows

---
# Windows 封装
进入引导界面后按下 `Ctrl + Shitf + F3`，系统会重启并跳过 OOBE。重启后可以直接用 Administrator 进入 Windows。

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

+ OpenSSH Server：[Releases · PowerShell/Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/releases)
+ Dism++：[Releases · Chuyu-Team/Dism-Multi-language](https://github.com/Chuyu-Team/Dism-Multi-language/releases)

### 跳过 TPM 检测
ESXi 平台上安装 Windows 11 提示无法满足系统要求，很可能是因为缺少 TPM 模块。有两种方式：修改注册表跳过 TPM 检测、创建 vTPM 模块（需要 vCenter）。

安装 Windows 11 时，引导界面显示选择语言项时，按住 `Shift + F10` 呼出命令行界面，输入以下命令配置注册表。

```bash
REG ADD HKLM\SYSTEM\Setup\LabConfig /v BypassTPMCheck /t REG_DWORD /d 1
```

