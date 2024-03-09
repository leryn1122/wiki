
# Jmeter 压力测试

## 常见问题
每次发起请求都会占用一个 Windows 压测机的本地端口，所有端口都被占用后 Jmeter 将报 `Address already in use` 的错误，因为所有端口都被占用而没有规划给操作系统。
调整 Windows 注册表两个参数，MaxUserPort 和 TCPTimedWaitDelay：

1. 新建 DWORD，名字为 `MaxUserPort`，输入数值 65534（十进制），确保所有端口都可以启用。
2. 新建 DWORD，名字为 `TCPTimedWaitDelay`，输入数值 30（十进制，单位秒，最小只能是 30 秒），确定 TCP/IP 可释放已关闭连接并重用其资源前，必须经过的时间，默认 240 秒。
3. 重启 Windows 服务器
```typescript
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\ /v MaxUserPort /d 65534 /t REG_DWORD /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\ /v TCPTimedWaitDelay /d 30 /t REG_DWORD /f
```
> Same user on each iteration：在 JMeter 中，User 就是线程，此选项的意思是说每个迭代都用相同的线程。
> 在以前 3.x 和 4.x 版本的 JMeter 中，是没有这个选项的。创建好 1 个线程后，每次迭代都是用这个线程，直到测试结束。它的影响就是，比如登录，加了 HTTP Cookie 管理器以后，单个线程多次迭代（注意不是多个线程哦）登录用的都是相同的 Cookie。

