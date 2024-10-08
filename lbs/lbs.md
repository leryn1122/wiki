---
id: lbs
tags:
- lbs
title: "\u8D1F\u8F7D\u5747\u8861"

---
# 负载均衡
负载均衡是一种网络技术，旨在多个计算机节点（或集群）、网络连接、CPU 和磁盘等等硬件资源上分配负载和资源，最优化资源配置、最大化吞吐量、最小化响应时间。

通常负载均衡和流量转发、路由转发具有一定程度的功能重叠。

## <font style="color:rgb(25, 27, 31);">负载均衡算法</font>
### <font style="color:rgb(25, 27, 31);">轮询 Round Robin</font>
<font style="color:rgb(25, 27, 31);">请求到达后，将客户端发送到负载均衡器的请求依次轮流地转发给服务集群的某个节点。</font>

<font style="color:rgb(25, 27, 31);">优点：实现简单，每个集群节点平均分担所有的请求。</font>

<font style="color:rgb(25, 27, 31);">缺点：当集群中服务器硬件配置不同、性能差别大时，无法区别对待。</font>

### <font style="color:rgb(25, 27, 31);">随机 Random</font>
<font style="color:rgb(25, 27, 31);">随机选取集群中的某个节点来处理该请求，由概率论的知识可知，随着请求量的变大，随机算法会逐渐演变为轮询算法，即集群各个节点会处理差不多数量的请求。</font>

<font style="color:rgb(25, 27, 31);">优点：简单使用，不需要额外的配置和算法。</font>

<font style="color:rgb(25, 27, 31);">缺点：随机数的特点是在数据量大到一定量时才能保证均衡，所以如果请求量有限的话，可能会达不到均衡负载的要求。</font>

### <font style="color:rgb(25, 27, 31);">加权 Weight</font>
<font style="color:rgb(25, 27, 31);">加权算法主要是根据集群的节点对应机器的性能的差异，给每个节点设置一个权重值，其中性能好的机器节点设置一个较大的权重值，而性能差的机器节点则设置一个较小的权重值。权重大的节点能够被更多的选中。它是和随机、轮训一起使用的。</font>

<font style="color:rgb(25, 27, 31);">优点：可以根据机器的具体情况，分配不同的负载，达到能者多劳。</font>

<font style="color:rgb(25, 27, 31);">缺点：需要额外管理加权系数。</font>

### <font style="color:rgb(25, 27, 31);">最小连接数</font>
<font style="color:rgb(25, 27, 31);">主要是根据集群的每个节点的当前连接数来决定将请求转发给哪个节点，即每次都将请求转发给当前存在最少并发连接的节点。</font>

<font style="color:rgb(25, 27, 31);">优点：可以根据集群节点的负载情况来进行请求的动态分发，即机器性能好，处理请求快，积压请求少的节点分配更多的请求。避免某个节点因为处理超过自身所能承受的请求量而导致宕机或者响应过慢。</font>

### <font style="color:rgb(25, 27, 31);">哈希 Hash</font>
<font style="color:rgb(25, 27, 31);">将对请求的 IP 地址或者 URL 计算一个哈希值，然后与集群节点的数量进行取模来决定将请求分发给哪个集群节点。它不是真正意义上的负载均衡，在某些意义上也是一个单点服务。</font>

<font style="color:rgb(25, 27, 31);">优点：实现简单</font>

<font style="color:rgb(25, 27, 31);">缺点：如果某个节点挂了，会使得一部分流量不可用。</font>

## 负载分类
### 硬件负载均衡
参考文档：

+ [应用和网络性能](https://www.f5.com.cn/solutions/application-network-performance)

用硬件厂商提供的专用负载均衡器。通常都是一体机，从专用硬件和驱动上实现负载均衡。

例如 F5 负载均衡器，其负载均衡能力高，但缺点是设备价格昂贵。据说一台 F5 负载均衡器的报价在 50W 左右。

![F5 负载均衡器架构](./../assets/1710605823203-4ef516d9-7c06-47a1-bdc8-90303dbec5ba.png)


### 软件负载均衡
利用应用层软件实现，例如 Nginx / LVS / HAProxy 等等。

+ [HAProxy & KeepAlived](https://www.yuque.com/leryn/wiki/lbs.haproxy)

### 应用层负载均衡
这里主要是应用层开发和维护自己的转发策略，例如 Spring Cloud API Gateway、Dubbo、<font style="color:rgb(25, 27, 31);">Spring Cloud Feign</font> 等等。

+ Spring Cloud API Gateway：
+ Dubbo：<font style="color:rgb(25, 27, 31);">支持4种算法（随机、轮询、活跃度、Hash一致性），而且算法里面引入权重的概念</font>
+ <font style="color:rgb(25, 27, 31);">Spring Cloud Feign：支持很多种算法，包括：轮询、随机、最小连接、区域加权、重试以及 ResponseTime 加权，也可以自己实现负载均衡算法。</font>

