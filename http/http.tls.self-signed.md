---
id: http.tls.self-signed
tags:
- http
- tls
title: "HTTPS \u81EA\u6388\u6743\u8BC1\u4E66"

---


# HTTPS 自授权证书
参考文档：

- [HTTPS 之 acme.sh 申请证书 - cnblog](https://www.cnblogs.com/clsn/p/10040334.html)
```bash
curl  https://get.acme.sh | sh
```
如果网络代理问题无法下载，直接查看脚本内的下载代码 `acme.sh`并手动安装即可。
```bash
 sh +x acme.sh --install-online
```
```bash
alias acme.sh=~/.acme.sh/acme.sh
echo "alias acme.sh='~/.acme.sh/acme.sh'" >> ~/.bash_aliases
```
```bash
# 注册邮箱
acme.sh --register-account -m pinktin@sina.com
```
安装过程中会自动为你创建 cronjob，每天 0:00 点自动检测所有的证书，如果快过期了，需要更新，则会自动更新证书。在该脚本的安装过程不会污染已有的系统任何功能和文件，所有的修改都限制在安装目录中：`~/.acme.sh/`
```bash
acme.sh --issue --dns -d 'leryn.top' -d '*.leryn.top' \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please --force
```
提示你在 DNS 服务器上增加两个条目, 然后重新执行命令并带上参数`--renew`
> TXT _acme-challenge uUH-miGK2Xm1pi3jJ81Wd2MYwILs3H5J1Eia2pfciKg
> TXT _acme-challenge DjR_Y27ab361PLJviE0hkjz2mTtJEIXlU-a5NAf6Kzk

```bash
acme.sh --issue --dns -d 'leryn.top' -d '*.leryn.top' \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please --force --renew
```
```bash
mkdir -p /etc/tls/leryn.top
cd /etc/tls/leryn.top

ln -s '/root/.acme.sh/*.leryn.top/fullchain.cer'   fullchain.cer
ln -s '/root/.acme.sh/*.leryn.top/*.leryn.top.key' leryn.top.key
```


### dns_api
各大 DNS 供应商 API 请参照：

- [https://github.com/acmesh-official/acme.sh/wiki/dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)
```bash
# 腾讯云 DNS Pod Token
export DP_Id="1234"
export DP_Key="sADDsdasdgdsf"

# 阿里云 RAM 
export Ali_Key="sdfsdfsdfljlbjkljlkjsdfoiwje"
export Ali_Secret="jlsdflanljkljlfdsaklkjflsa"

acme.sh --issue --dns -d 'leryn.top' -d '*.leryn.top'
```
