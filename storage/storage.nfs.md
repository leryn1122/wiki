<a name="wnBSI"></a>
# NFS
NFS（Network File System）表示层协议。<br />需要开通网络权限的请注意：

- NFS 网络默认使用了 111 和 2049 端口的 TCP/UDP 协议，并且 UDP 是双向的。请同时开通**两个端口**的 TCP 以及**双向 UDP**。
<a name="Y8koD"></a>
## 服务端安装
**Ubuntu 系统**
```bash
sudo apt install nfs-kernel-server
```
**CentOS 系统**
```bash
sudo yum install nfs-utils rpcbind
sudo systemctl start nfs-server.service
```
<a name="jkqL9"></a>
## 客户端安装
**Ubuntu 系统**
```bash
sudo apt install nfs-common
```
**CentOS 系统**
```bash
sudo yum install nfs-utils
```
<a name="czF0G"></a>
## 挂载
首先需要在服务端配置：

- exports 的相关配置请参考文档：[https://www.cnblogs.com/st-jun/p/7742560.html](https://www.cnblogs.com/st-jun/p/7742560.html)
```bash
vim /etc/exports

/path/to/mnt 196.168.0.0/24(rw,sync,insecure,no_subtree_check,no_all_squash,no_root_squash)

# 重新加载配置，不会导致已有挂载断开
exportfs -rv
```
挂载命令：
```bash
# 创建挂载目录
mkdir -p /mnt/path

# 用NFS挂载
sudo mount -t nfs -o vers=3,tcp,nolock,async,mountproto=tcp,rsize=1048576,wsize=1048576  121.196.30.0:/path/to/mnt /mnt/path
```
<a name="ffY9M"></a>
## Windows 客户端支持
<a name="0a26c4f7"></a>
#### 第一步：配置远程桌面服务

1. 打开【服务器管理器】，点击【添加角色和功能】
2. 选择【基于角色或基于功能的安装】，默认选项即可→【远程桌面服务】默认不用操作，【桌面会话主机】【远程桌面授权 】→【添加功能】完成安装
3. 在 cmd 中输入`gpedit.msc`，打开计算机本地组策略。
4. 在计算机本地组策略里选择【计算机配置 】→【管理模板-windows 组件 】→【远程桌面服务 】→【远程桌面会话主机 】→【连接】，<br />找到 【限制限制连接数量（可根据具体数量设置）】和【将远程桌面服务用户限制到单独的远程桌面服务会话（禁用）】<br />【这台电脑 】→【属性 】→【远程设置】，选择【允许远程连接到此计算机】，去掉下方【仅允许允许使用网络级别身份验证的……】的勾，<br />【选择用户】，输入远程连接的用户，点击确定

参考地址：

- [https://jingyan.baidu.com/article/22fe7ced1e696d7102617ff5.html](https://jingyan.baidu.com/article/22fe7ced1e696d7102617ff5.html)
- [https://jingyan.baidu.com/article/9faa7231c9e061473d28cb65.html](https://jingyan.baidu.com/article/9faa7231c9e061473d28cb65.html)
- [https://www.52sanmiao.com/xitongyw/49.html](https://www.52sanmiao.com/xitongyw/49.html)
<a name="c6006f86"></a>
#### 第二步：配置 Windows 系统 NFS 客户端服务

1. 启动 Windows NFS 客户端服务：
> - 打开【控制面板】→【程序】→【打开或关闭 Windows 功能】→【NFS 客户端】
> - 勾选 NFS 客户端，即开启 Windows NFS 客户端服务
> - 在 Windows Server 2012 中，NFS 客户端在【文件和存储服务】→【文件和 ISCSI 服务】下，点击安装即可

2. 挂载网络驱动器：
```bash
mount ***.***.***.***:/path/to/mnt X:
```
成功挂载，打开我的电脑，可以在网络位置看到 `X:` 盘了

3. 取消挂载<br />直接在【我的电脑】里面鼠标点击取消映射网络驱动器 `X:`
```bash
umount X:

# 或者取消所有网络驱动器
umount -a
```
<a name="hBevZ"></a>
### 常见问题
**Windows Server 2016 NFS 挂载成功后，没有写权限**

1. 点击挂载虚拟盘，打开属性，查看 nfs 装载选项，uid 和 gid 为 -2 时，修改注册表，跳转到`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default` 目录下新建两个 DWORD 值 ，名称为：`AnonymousGid` 和 `AnonymousUid`，数据值为 0 和 2。确认数据都保存以后，系统重启。
```bash
# 管理员权限下开启命令行, 执行如下两个命令并重启机器
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default\ /v AnonymousUid /d 0 /t REG_DWORD /f
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default\ /v AnonymousGid /d 0 /t REG_DWORD /f
```
**Windows 10 NFS 挂载成功后，没有写权限**<br />Win10 系统就是修改 `/etc/export` 文件中的数据，添加 anongid 与 anonuid
```bash
/path/to/mnt 196.168.0.0/24(rw,sync,insecure,no_subtree_check,all_squash,anonuid=0,anongid=0)
```

