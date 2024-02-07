
# Redis 介绍

#### maxmemory

我们可以通过配置 `redis.conf` 中的 maxmemory 这个值来开启内存淘汰功能. maxmemory 为 0 的时候表示我们对 Redis 的内存使用没有限制

Redis提供了下面几种淘汰策略供用户选择, 其中默认的策略为 noeviction 策略:

|  |  |
| --- | --- |
| noeviction | 当内存使用达到阈值的时候, 所有引起申请内存的命令会报错 |
| allkeys-lru | 在主键空间中, 优先移除最近未使用的 key |
| volatile-lru | 在设置了过期时间的键空间中, 优先移除最近未使用的 key |
| volatile-lru | 在设置了过期时间的键空间中, 优先移除最近未使用的 key |
| allkeys-random | 在主键空间中, 随机移除某个 key |
| volatile-random | 在设置了过期时间的键空间中, 随机移除某个 key |
| volatile-ttl | 在设置了过期时间的键空间中, 具有更早过期时间的 key 优先移除 |


#### maxmemory-policy noeviction

Redis 也支持 Runtime 修改淘汰策略, 这使得我们无需重启 Redis 实例而实时的调整内存淘汰策略<br />下面看看几种策略的适用场景:

|  |  |
| --- | --- |
| allkeys-lru | 如果我们的应用对缓存的访问符合幂律分布 (也就是存在相对热点数据) 或者我们不太清楚我们应用的缓存访问分布状况, 我们可以选择 allkeys-lru 策略 |
| allkeys-random | 如果我们的应用对于缓存key的访问概率相等, 则可以使用这个策略 |
| volatile-ttl | 这种策略使得我们可以向 Redis 提示哪些 key 更适合被 eviction |

