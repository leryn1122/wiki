<a name="Ij1eF"></a>
# DNS
阿里云购买域名后自动解锁对应域名的 DNS 服务. 域名购买按年付费, 域名架构根据不同域名, 价格不同.<br />当前使用的`leryn.top`域名约 10-30 人民币左右.
<a name="TSZ9P"></a>
## 备案
国内公网使用域名需要工信部备案, 否则无法在公网开通 80, 443 端口的服务 (~~内网可无视~~). 客服小姐姐也会在域名备案时提供人工服务. 域名备案后, 建议尽量在主页的最下方展示域名备案.
<a name="ETTua"></a>
## Https 证书
此外, 阿里云可以免费领取 20 个域名证书, 但仅支持单域名, 不支持泛域名. 使用泛域名, 推荐使用`Let's Encryption`生成自授权证书. 具体见 [HTTPS 自授权证书](https://www.yuque.com/leryn/wiki/https).
<a name="YDLN1"></a>
## DNS配置

- `*`表示泛域名;
- `@`表示空域名;
- `_acme-challenge`用于`acme.sh`域名认证;
| 主机 | 记录类型 | 值 |
| --- | --- | --- |
| * | CNAME | leryn.top |
| www | CNAME | leryn.top |
| @ | A | IP 地址 |
| _acme-challenge | TXT | 授权码 |

<a name="SG80L"></a>
### hosts 文件

这里由于我是单服务器, 没有集群, 所以可以使用本地 DNS 配置 `/etc/hosts`, 将域名指向 localhost. 流量会先走本地 DNS, 并且是环回地址, 不会从外部网卡出网, 可以大幅缩短网络 IO 时间.<br />**请不要**在公司中使用这种方法, 这会导致服务器的配置混乱, 无法管理.

```bash
sudo vim /etc/hosts
```

```
127.0.0.1 leryn.top
127.0.0.1 *.leryn.top
```

Linux 和 Windows 对应的 hosts 文件地址:
```
/etc/hosts
C:\Windows\System32\drivers\etc\hosts
```

