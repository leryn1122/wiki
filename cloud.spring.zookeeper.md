


```bash
docker run --rm zookeeper:latest bash -c cp -ar /conf/* /conf/zookeeper
```
```bash
docker run \
  --detach=true \
  --publish=2181:2181 \
  --privileged=true \
  --volume=/conf/zookeeper:/conf \
  --volume=/data/zookeeper:/data \
  --name=zookeeper \
  --hostname=zookeeper \
  zookeeper:latest
```



<a name="XWJ8R"></a>
## zoo.cfg

```bash
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
standaloneEnabled=true
admin.enableServer=true
server.1=localhost:2888:3888;2181
```
