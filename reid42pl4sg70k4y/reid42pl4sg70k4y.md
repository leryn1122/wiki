Jmeter



> Same user on each iteration：在 JMeter 中，User 就是线程，此选项的意思是说每个迭代都用相同的线程。
> 在以前 3.x 和 4.x 版本的 JMeter 中，是没有这个选项的。创建好 1 个线程后，每次迭代都是用这个线程，直到测试结束。它的影响就是，比如登录，加了 HTTP Cookie 管理器以后，单个线程多次迭代（注意不是多个线程哦）登录用的都是相同的 Cookie。


（1）右键Parameters 新建DWORD，名字为MaxUserPort,输入数值65534（十进制）<br />（2）再次右键 Parameters 新建DWORD，名字为TCPTimedWaitDelay,输入数值0（十进制）表示立刻回收端口
```typescript
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
```
