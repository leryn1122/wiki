参考文档:

- [API文档 - 官网](https://docs.docker.com/engine/api/)
- [非官方SDK - 官网](https://docs.docker.com/engine/api/sdk/#unofficial-libraries)

很不幸, docker 官方只提供了 go, python 和 HTTP 三种官方接口. 但是他罗列了一些非官方 SDK.
<a name="mc8mc"></a>
# Docker API 接口
<a name="RfbTE"></a>
## Docker 命令行
<a name="dzZvh"></a>
### 查看 docker 存储空间

```bash
# 查看空间
docker system df

# 命令可以进一步查看空间占用细节, 以确定是哪个镜像、容器或本地卷占用过高空间
docker system df -v
```
<a name="DDtB9"></a>
### 清理空间

- 已停止的容器: 指所有已被容器 (包括 stop 的) 关联的镜像, 也就是 docker ps -a 所看到的所有容器对应的 image
- 未被引用的镜像: 没有被分配或使用在容器中的镜像
- 未被任何容器使用的卷
- 未被任何容器所关联的网络
- 所有悬空的镜像: 未配置任何 Tag (也就是无法被引用) 的镜像. 通常是由于镜像编译过程中未指定 -t 参数配置 Tag 导致
```bash
#
docker system prune
# 一并清除所有未被使用的镜像和悬空镜像
docker system prune -a
# 用以强制删除, 不提示信息
docker system prune -f
# 删除悬空的镜像
docker image prune
# 删除无用的容器
docker container prune
#
docker container prune --filter "until=24h"
# 删除无用的卷
docker volume prune
# 删除无用的网络
docker network prune
# 
docker builder prune
```
<a name="jHTDC"></a>
### 手动清除

对于悬空镜像和未使用镜像可以使用手动进行个别删除:

```bash
# 删除所有悬空镜像, 不删除未使用镜像:
docker rmi $(docker images -f "dangling=true" -q)

# 删除所有未使用镜像和悬空镜像
docker rmi $(docker images -q)

# 清除一些不使用的卷
docker volume rm $(docker volume ls -qf dangling=true)

# 删除所有已退出的容器：
docker rm -v $(docker ps -aq -f status=exited)

# 删除所有状态为 dead 的容器
docker rm -v $(docker ps -aq -f status=dead)
```
<a name="pHuLb"></a>
### 批量操作

```bash
# 更新所有latest镜像
docker images --format "{{.Repository}}:{{.Tag}}" | grep ':latest' | xargs -L1 docker pull

# 更新所有镜像
docker images --format "{{.Repository}}:{{.Tag}}" | xargs -L1 docker pull

# 强制删除空镜像
docker images | awk '$1=="<none>"||$2=="<none>"{print $3}' | xargs -r docker rmi --force
```
<a name="ICNj9"></a>
## Docker HTTP 接口

出于安全性考虑, 本人不推荐在没有配置 TLS 前暴露 HTTP 接口.
<a name="E4Bm4"></a>
### 远程构建镜像

需要将本地项目打包成 tar 包, 再使用 HTTP 工具发送到远端. tar 包解压出的文件夹将作为 docker 构建的上下文路径. 甚至可以将这个方法包装为前端项目的构建脚本, 而不用流水线.

```bash
curl -XPOST 'http://xxx.xxx.xxx.xxx:2375/build?t=image:tag' \
     -H 'application/x-tar' \
     --data-binary @app.tar.gz
```

<a name="Ax22t"></a>
# Docker Registry API 接口
简易版
```bash
docker exec registry rm -rf /var/lib/registry/docker/registry/v2/repositories/<image>
docker exec registry bin/registry garbage-collect /etc/docker/registry/config.yml
docker restart registry
```

