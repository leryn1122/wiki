---
id: nosql.redis.intro
tags: []
title: "Redis \u4ECB\u7ECD"

---
# Redis 介绍
## 过期时间 和 内存驱逐策略
  
Redis 是缓存中间件，不要把 Redis 当数据库使用，Redis 没有的值请去数据库加载

Redis 的 key 都推荐设置一个过期时间

如果同时加载了大量的 Key 到 Redis 且设置了相同的过期时间，那么大量 Key 会在同一时间过期。又因为 Redis 是单线程的，Redis 回收过期的 Key 出现阻塞，会导致读写也阻塞。

这种情况下应当使用 unlink 让 Redis 在空闲的时候回收过期的 Key，且给 Key 的过期时间追加一个随机的值，让这些 Key 不在同一时间过期

#### maxmemory


我们可以通过配置 `redis.conf` 中的 maxmemory 这个值来开启内存淘汰功能. maxmemory 为 0 的时候表示我们对 Redis 的内存使用没有限制



Redis提供了下面几种淘汰策略供用户选择, 其中默认的策略为 noeviction 策略:

| | |
| --- | --- |
| noeviction | 当内存使用达到阈值的时候, 所有引起申请内存的命令会报错 |
| allkeys-lru | 在主键空间中, 优先移除最近未使用的 key |
| volatile-lru | 在设置了过期时间的键空间中, 优先移除最近未使用的 key |
| volatile-lru | 在设置了过期时间的键空间中, 优先移除最近未使用的 key |
| allkeys-random | 在主键空间中, 随机移除某个 key |
| volatile-random | 在设置了过期时间的键空间中, 随机移除某个 key |
| volatile-ttl | 在设置了过期时间的键空间中, 具有更早过期时间的 key 优先移除 |


#### maxmemory-policy noeviction


Redis 也支持 Runtime 修改淘汰策略, 这使得我们无需重启 Redis 实例而实时的调整内存淘汰策略

下面看看几种策略的适用场景:

| | |
| --- | --- |
| allkeys-lru | 如果我们的应用对缓存的访问符合幂律分布 (也就是存在相对热点数据) 或者我们不太清楚我们应用的缓存访问分布状况, 我们可以选择 allkeys-lru 策略 |
| allkeys-random | 如果我们的应用对于缓存key的访问概率相等, 则可以使用这个策略 |
| volatile-ttl | 这种策略使得我们可以向 Redis 提示哪些 key 更适合被 eviction |




## Springboot 支持


Springboot 默认使用 Lettuce，如果 Classpath 中有 Jedis 则切换为 Jedis。

可选 Lettuce、Jedis 作为底层客户端的实现类，推荐使用 Lettuce + Jedission 具有更好的性能，功能更加强大：[https://docs.spring.io/spring-data/redis/reference/redis/drivers.html](https://docs.spring.io/spring-data/redis/reference/redis/drivers.html)

+ Lettuce：支持同步/异步 API，线程安全，共享 Redis 连接
+ Jedis：只支持同步 API，线程不安全，需要额外引入 Jedis 线程池



```xml
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/io.lettuce/lettuce-core -->
        <dependency>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.redisson/redisson-spring-boot-starter -->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson-spring-boot-starter</artifactId>
            <version>3.48.0</version>
        </dependency>
    </dependencies>
```

配置文件：

```yaml
# 哨兵模式
spring:
  redis:
    password: xxx
    sentinel:
      master: xxx
      nodes: xxx.xxx.xxx.xxx:26379,xxx.xxx.xxx.xxx:26379,xxx.xxx.xxx.xxx:26379
      password: xxx
    database: 0
```

```yaml
# 集群模式
spring:
  redis:
    password: xxx
    clsuster:
      nodes:
        - xxx.xxx.xxx.xxx:6379
        - xxx.xxx.xxx.xxx:6379
        - xxx.xxx.xxx.xxx:6379
        - xxx.xxx.xxx.xxx:6379
        - xxx.xxx.xxx.xxx:6379
        - xxx.xxx.xxx.xxx:6379
    database: 0
    lettuce:
      cluster:
        refresh:
          period: 20s
          adaptive: true
```

