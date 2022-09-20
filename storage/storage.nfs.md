<a name="wnBSI"></a>
# NFS
NFS (Network File System) 表示层协议

挂载命令:

```bash
# 创建挂载目录
mkdir -p /mnt/path

# 用NFS挂载
sudo mount -t nfs -o vers=3,tcp,nolock,async,mountproto=tcp,rsize=1048576,wsize=1048576  127.0.0.1:/path/to/mnt /mnt/path
```

<a name="5c706029"></a>
## NFS Windows 客户端连接
<a name="Iahuh"></a>
### Windows 客户端远程挂载 nfs

<a name="47a5ed21"></a>
#### nfs 服务器端的安装(ubuntu)

> 1.安装 nfs server<br />执行命令： sudo apt update<br />sudo apt install nfs-kernel-server<br />2.安装成功后，查看 nfs 版本信息<br />cat /proc/fs/nfsd/versions<br />3.修改 nfs 配置文件<br />vim /etc/exports<br />4.修改内容类似于下：<br />/home 10.65.137.0/24(rw,sync,insecure,no_subtree_check,no_all_squash,no_root_squash)<br />/home 10.0.0.0/255.0.0.0(rw,sync,insecure,no_subtree_check,all_squash,anonuid=995,anongid=1001) （anonuid 和 anongid 是制定哪个用户）<br />exports 的相关配置请参考文档：[https://www.cnblogs.com/st-jun/p/7742560.html](https://www.cnblogs.com/st-jun/p/7742560.html) <br />5.重新加载 nfs 配置文件<br />exportfs -rv


<a name="0a26c4f7"></a>
#### 第一步：配置远程桌面服务

> 打开“服务器管理器”，点击“添加角色和功能”<br />选择基于角色或基于功能的安装，点击下一步<br />默认选项即可，点击下一步<br />选择远程桌面服务，点击下一步<br />默认不用操作，继续点击下一步<br />选择“桌面会话主机”和“远程桌面授权 ”，在弹出的窗口中点击“添加功能”，点击下一步。<br />继续点击下一步,完成安装<br />在 cmd 中输入“gpedit.msc”，打开计算机本地组策略。<br />在计算机本地组策略里选择计算机配置-管理模板-windows 组件-远程桌面服务-远程桌面会话主机-连接，<br />找到 “限制限制连接数量（可根据具体数量设置）”和“将远程桌面服务用户限制到单独的远程桌面服务会话（禁用）”<br />这台电脑--属性--远程设置，选择“允许远程连接到此计算机”，去掉下方“仅允许允许使用网络级别身份验证的……”的勾，<br />“选择用户”，输入远程连接的用户，点击确定
>  
> 参考地址：[https://jingyan.baidu.com/article/22fe7ced1e696d7102617ff5.html](https://jingyan.baidu.com/article/22fe7ced1e696d7102617ff5.html)<br />[https://jingyan.baidu.com/article/9faa7231c9e061473d28cb65.html](https://jingyan.baidu.com/article/9faa7231c9e061473d28cb65.html)<br />[https://www.52sanmiao.com/xitongyw/49.html](https://www.52sanmiao.com/xitongyw/49.html)


<a name="c6006f86"></a>
#### 第二步：配置 windows 系统 nfs 客户端

> 启动 windows Nfs 客户端服务：<br />**1.打开控制面板->程序->打开或关闭 windows 功能->NFS 客户端**<br />勾选 NFS 客户端,即开启 windows NFS 客户端服务.<br />在 windows Server2012 中，nfs 客户端在“文件和存储服务”--“文件和 ISCSI 服务”下，点击安装即可<br />**2.win+R->cmd**<br />**mount 10.65.137.57:/exports/nfs/news_source/research X:**<br />成功挂载，打开我的电脑，可以在网络位置看到 X：盘了<br />解释:<br />mount,是指令<br />10.65.137.57 你的 linux 主机 IP<br />/exports/nfs/news_source/research 你的共享目录<br />X:你挂载的网络文件盘--注意,可能会与你的其他盘冲突,你可以随意更改<br />**3.取消挂载**<br />直接在 我的电脑 里面鼠标点击取消映射网络驱动器 X:<br />或者: win+R->cmd<br />输入: umount X:<br />(umount -a 取消所有网络驱动器)


<a name="41d0eebc"></a>
#### linux 环境下 nfs 客户端安装(ubuntu)

> 1.虚拟机需要先安装 nfs 客户端<br />sudo apt-get install nfs-common 安装 nfs client<br />2.使用 mount 命令进行挂载 nfs 服务<br />sudo mount -t nfs 192.168.2.2:/export/nfs/news /export/nfs


<a name="80da6145"></a>
#### linux 环境下 nfs 客户端安装(centos)

```bash
sudo yum -y install nfs-utils
```

<a name="088e3640"></a>
#### windowsServer2016 nfs 挂载成功后，没有写权限

> 1.点击挂载虚拟盘，打开属性，查看 nfs 装载选项，uid 和 gid 为-2 时，修改注册表，跳转到<br />HKEY_LOCAL_MACHINE \ SOFTWARE \ Microsoft \ ClientForNFS \ CurrentVersion \ Default 目录下<br />新建两个 DWORD（32 位）值 ，名称为：AnonymousGid 和 AnonymousUid,数据值为 0 2.确认数据都保存以后，在超融合上将系统重启


```
# 管理员权限下开启命令行, 执行如下两个命令并重启机器
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default\ /v AnonymousGid /d 0 /t REG_DWORD /f
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default\ /v AnonymousUid /d 0 /t REG_DWORD /f
```

<a name="53f8b432"></a>
#### windows10 nfs 挂载成功后，没有写权限

> win10 系统就是修改/etc/export 文件中的数据，添加 anongid 与 anonuid<br />eg: /export/nfs 10.65.138.**/24(rw,sync,insecure,no_subtree_check,all_squash,anonuid=0,anongid=0)<br />nfs 的 exports 配置文件参数参考文件：[https://www.cnblogs.com/st-jun/p/7742560.html](https://www.cnblogs.com/st-jun/p/7742560.html)


<a name="4b31e3cb"></a>
#### ubuntu 客户端 nfs 服务异常

> 查看服务状态 systemctl status nfs-common<br />如果状态异常，执行以下三条命令


```
cd /lib/systemd/system/
mv nfs-common.service nfs-common.service.bak
systemctl daemon-reload
```
