Nexus 可以支持 Docker Registry, npm, maven, maven android, nuget 多种仓库.
<a name="zonc3"></a>
# Nexus 安装手册
一般服务器的配置可能不够搭建 Nexus, 我司的 Nexus 都是使用实体机才撑得起服务.
<a name="psNe6"></a>
## Docker 安装
```bash
docker run \
  --detach=true \
  --env=NEXUS_CONTEXT=nexus \
  --privileged=true \
  --publish=8083:8081 \
  --restart=always \
  --volume=/data/nexus:/nexus-data \
  --name=nexus \
  --hostname=nexus \
  docker.io/sonatype/nexus3:latest
```

配置`repository-list.txt`:

```
# maven-aliyun
http://maven.aliyun.com/nexus/content/groups/public
# maven-apache-snapshot
https://repository.apache.org/content/repositories/snapshots/
# maven-apache-release
https://repository.apache.org/content/repositories/releases/
# maven-atlassian
https://maven.atlassian.com/content/repositories/atlassian-public/
# maven-central.maven.org
http://central.maven.org/maven2/
# maven-datanucleus
http://www.datanucleus.org/downloads/maven2
# maven-central
https://repo1.maven.org/maven2/
# nexus.axiomalaska.com
http://nexus.axiomalaska.com/nexus/content/repositories/public
# oss.sonatype.org
https://oss.sonatype.org/content/repositories/snapshots
# pentaho
https://public.nexus.pentaho.org/content/groups/omni/
```
