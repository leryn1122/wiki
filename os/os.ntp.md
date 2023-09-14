
# NTP 服务

参考文档:

- [Linux的NTP配置总结 - 潇湘隐者 - 博客园](https://www.cnblogs.com/kerrycode/p/4744804.html)
- [windows时间同步设置 - Dark_Elf - 博客园](https://www.cnblogs.com/zhangdongyu/p/15674265.html)
- [如何在Windows10上安装NTP客户端更精确的校准系统时间](https://blog.minirplus.com/8808/)

## 简介
> 下面是网上关于 ntpd 与 ntpdate 区别的相关资料。如下所示所示。
> 使用之前得弄清楚一个问题，ntpd 与 ntpdate 在更新时间时有什么区别。`ntpd` 不仅仅是时间同步服务器，它还可以做客户端与标准时间服务器进行同步时间，而且是平滑同步，并非 ntpdate 立即同步，在生产环境中慎用 ntpdate，也正如此两者不可同时运行。
> 时钟的跃变，对于某些程序会导致很严重的问题. 许多应用程序依赖连续的时钟——毕竟，这是一项常见的假定，即取得的时间是线性的，一些操作，例如数据库事务，通常会地依赖这样的事实：时间不会往回跳跃。不幸的是，ntpdate 调整时间的方式就是我们所说的“跃变”：在获得一个时间之后，ntpdate 使用 settimeofday(2) 设置系统时间，这有几个非常明显的问题。
> - 这样做不安全。ntpdate 的设置依赖于 ntp 服务器的安全性，攻击者可以利用一些软件设计上的缺陷，拿下 ntp 服务器并令与其同步的服务器执行某些消耗性的任务。由于 ntpdate 采用的方式是跳变，跟随它的服务器无法知道是否发生了异常（时间不一样的时候, 唯一的办法是以服务器为准）。
> - 这样做不精确。一旦 ntp 服务器宕机，跟随它的服务器也就会无法同步时间。与此不同，ntpd 不仅能够校准计算机的时间，而且能够校准计算机的时钟。
> - 这样做不够优雅。由于是跳变，而不是使时间变快或变慢，依赖时序的程序会出错（例如如果 ntpdate 发现你的时间快了，则可能会经历两个相同的时刻，对某些应用而言，这是致命的）。因而唯一一个可以令时间发生跳变的点，是计算机刚刚启动，但还没有启动很多服务的那个时候。其余的时候，理想的做法是使用 ntpd 来校准时钟，而不是调整计算机时钟上的时间。
> 
ntpd 在和时间服务器的同步过程中，会把 BIOS 计时器的振荡频率偏差——或者说 Local Clock 的自然漂移(drift)——记录下来。这样即使网络有问题，本机仍然能维持一个相当精确的走时。


## 安装前的注意点
注意点：

- `ntp`和`ntpdate`都占用了`123`端口，如果`ntp`的 daemon 进程启动时, 无法使用`ntpdate`。
- Ubuntu 默认安装了`chrony`服务，服务器重启后会默认启用这个服务，导致 NTP 无法自启动。虽然`chrony`比`ntp`更加先进，但很多公司仍然在使用 ntp。
- `ntp`使用了双向 UDP 协议校时，网络权限需要放开 UDP 协议。

## 包管理器安装
安装 NTP 服务：
```bash
sudo rpm -qa | grep ntp

sudo apt install -y ntp ntpstat
```
修改 NTP 的配置文件：
```bash
vim /etc/ntp.conf

# 注释所有第三方的ntp服务器
# server 0.centos.pool.ntp.org iburst
# server 1.centos.pool.ntp.org iburst
# server 2.centos.pool.ntp.org iburst
# server 3.centos.pool.ntp.org iburst
# server 127.127.1.0

# 在文档末尾添加如下
# 注意Fudge需要首字母大写
# 在文档末尾添加如下
server xxx.xxx.xxx.xxx
Fudge xxx.xxx.xxx.xxx stratum 3
```
启动服务：
```bash
# 启动服务
systemctl start ntpd

# 设置开机自启
systemctl enable ntpd

# 查看状态
systemctl status ntpd

# 每次修改配置文件后都需要重启服务
systemctl restart ntpd
```
用命令查看当前`ntp`同步状态。`ntp`服务启动后需要等待 5-10 分钟左右才能和`ntp`服务器同步，有时候要更久。如果显示`synchronised`说明已经同步完成。
```bash
ntpstat

# 说明同步尚未完成
unsynchronised
  time server re-starting
   polling server every 8 s
# 说明同步已经完成
synchronised to NTP server (xxx.xxx.xxx.xxx) at stratum .
   time correct to within 26 ms
   polling server every 64 s
```
`ntpq`查看同步上层状态信息
```bash
# 查看同步主机
ntpq -p


# 结果
remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*xxx.xxx.xxx.xxx xxx.xxx.xxx.xxx  2 u   17   64  377   32.823   -0.201   2.141
 xxx.xxx.xxx.xxx .STEP.          16 u    - 1024    0    0.000    0.000   0.000
```
| 表头字段名 | 含义 |
| --- | --- |
| remote | 用于同步的远程节点或服务器. “LOCAL”表示本机 (当没有远程服务器可用时会出现) |
| refid | 远程的服务器进行同步的更高一级服务器 |
| st | 远程节点或服务器的 Stratum(级别, NTP 时间同步是分层的) |
| t | 类型 (u: unicast(单播) 或 manycast(选播) 客户端, b: broadcast(广播) 或 multicast(多播) 客户端, l: 本地时钟, s: 对称节点(用于备份), A: 选播服务器, B: 广播服务器, M: 多播服务器, 参见“Automatic Server Discovery“) |
| when | 最后一次同步到现在的时间 (默认单位为秒, “h”表示小时, “d”表示天) |
| poll | 同步的频率: RFC 5905 建议在 NTPv4 中这个值的范围在 4 (16 秒) 至 17 (36 小时) 之间(即 2 的指数次秒), 然而观察发现这个值的实际大小在一个小的多的范围内 : 64 (26 )秒 至 1024 (210 )秒 |
| reach | 一个 8 位的左移移位寄存器值, 用来测试能否和服务器连接, 每成功连接一次它的值就会增加, 以 8 进制显示 |
| delay | 从本地到远程节点或服务器通信的往返时间(毫秒) |
| offset | 主机与远程节点或服务器时间源的时间偏移量, offset 越接近于 0, 主机和 NTP 服务器的时间越接近(以方均根表示, 单位为毫秒) |
| jitter | 与远程节点同步的时间源的平均偏差(多个时间样本中的 offset 的偏差, 单位是毫秒), 这个数值的绝对值越小, 主机的时间就越精确 |

