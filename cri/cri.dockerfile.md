
# Dockerfile 最佳实践

参考文档：

- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Overview of best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

Dockerfile 是一个用来构建镜像的唯一方式，这很基本也很简单，重点是优化。如果需要完整的学习 Dockerfile，请跳转到官方文档。

## Dockerfile 优化
糟糕的 Dockerfile 可能会构建一个 1-2G 的镜像，这是不能接受的。较大的镜像无论是在流水线构建还是环境部署时都是非常耗时耗资源的。
```bash
# 如何查看镜像大小
# SIZE列
docker images

# 第一块表的SIZE, SHARED SIZE, UNIQUE SIZE三列
docker system df --verbose
```
通常镜像都不会特别大，一般不会超过 500-600MB，除非项目本身特别大（`rancher/rancher:v2.6.2`大小在 1.12GB）。

- C++ 和 go 二进制镜像可能会比较小， 100-200MB，甚至 ≤100MB。
- Java 镜像包含了 JVM 运行时，所以可能会稍大，500-600MB，JDK 本身约 400MB。

如果镜像超出以上参考值，可能需要优先 Dockerfile。

### 写在前面
Dockerfile 的每一个命令，都会独立的形成一个 UnionFS 文件层。保存了与上一层的差别文件。但只有`RUN`，`COPY`，`ADD`会产生有大小的中间层。所以使用多句命令创建并删除文件，这些文件依旧存在于之前层中，只是对当前层不可见了，最终镜像的体积将是这些层体积的叠加。
分层的另一个好处是利于分发，镜像相同的层将在同一节点上共用。拉取新镜像时，共用的层如果已经存在于节点上，将不会被重新拉取。这既可以节省网络流量、也可以节省节点本地磁盘的占用。
如何查看层的历史：
```bash
docker history <IMAGE>
```

### .dockerignore
类似`.gitignore`、`.eslintignore`等等，排除与 Dockerfile 无关的文件，语法也几乎一致。这些文件仍然会出现在构建上下文当中，但无法使用`COPY`或`ADD`加入镜像。
以下文件应当被忽略：

- .git 包含了全部代码历史，包含敏感信息而且非常大
- 与构建无关 或 可以被当前项目构建出的文件
- 体积较大
- 包含敏感信息的文件
- 可以用 stdin 写入的文件，可以直接用`echo EOF`的方式写在 Dockerfile 中

### 解耦应用程序
这步跟 Dockerfile 都无关，从代码和仓库架构上应该解耦应用。将应用程序拆成不同仓库、不同的镜像，而不是将前后端，对客端和管理端全部糅杂在一个超级镜像中。

### 更小的基镜像
:::info
镜像需要精简但请至少保证可调试
:::
请使用更小的基镜像来构建，例如`slim`、`alpine`镜像。这些镜像为了追求极致的小：

- 利用`busybox`、`alpine`等裁剪过的基镜像来构建
- 通常不会包含一些不必要的命令，甚至可能没有 bash，但这些命令对于构建和运行来说不一定是必要的。尽管很多包很有用，但额外或不必要的包需要出现在镜像中。例如数据库镜像中并不需要文本编辑器。
- 更小的 runtime，例如：Java 只使用 JRE，而不是 JDK

有些小公司可能会将一些无关的工具打包到镜像中，例如一些 hack 的 python 脚本

| Image | Size |
| --- | --- |
| openjdk:17.0.1-jdk-buster | 306.43 MB |
| openjdk:17.0.1-jdk-slim | 210.27 MB |


### 使用内网镜像源
这不是 Docker 的技巧。请尽量使用内网或者国内镜像源（容器或制品库），来提高下载依赖包时的速度，同时保证依赖的安全性。
例如：

- npm 的`.npmrc`
- Maven 的`settings.xml`
- 配置内网`GO_PROXY`

### 使用中间层缓存
如果没有使用`--no-cache`选项，那么 Dockerfile 中的指令都会存在缓冲层，下次如果遇到同样的结果就会直接调用缓冲层，以提高性能和节省磁盘空间。
启用缓存的因素主要有两点：

- 构建上下文是否变化：如果上下文中存在文件发生变化，即使这个文件不影响构建，这些增量变化拷贝到了上下文中（例如`COPY . .`）也会导致缓存失效。所以无关的文件不拷贝到上下文或推迟到需要使用时再拷贝或用 .dockerignore 排除无关的文件。
- 命令是否会从网络加载资源：如果 Dockerfile 命令产生了从外部网络加载资源，不能保证相同的命令能加载到同样的资源，所以构建也不会使用缓存。

这样就需要将更通用的命令分开并写在更前面，差异化的命令靠后。
例如：

- 前文的复制`.npmrc`和`settings.xml`这一步会直接使用缓存
- 非常长的`apt-get install -y`

### 使用单句命令
既然多句 Dockerfile 会产生非常多的中间层，那么索性尽量把命令都用`&&`拼凑到一行命令上。大量官方镜像的命令都使用了这个技巧，每个`RUN`命令可能都有上百行。
例如如下命令，排序后可以优化安装顺序。同时也易于后期修改和代码评审。
```dockerfile
RUN apt-get update && apt-get install -y \
      bzr \
      cvs \
      git \
      mercurial \
      subversion \
 && rm -rf /var/lib/apt/lists/*
```

### 外部构建
通常基础应用的镜像都会选择这种方式构建，大多`FROM scratch`镜像。
与容器内构建不同，有的人选择外部构建成 tarball 包，因此 Dockerfile 退化为验签并解压 tarball 包。可以参考 OpenJDK 等等官方镜像都是这么编写的，将构建好的 JDK 以 tarball 的形式直接 COPY 到容器内，并比较 SHA256 信息。

### 多阶段构建
多阶段构建将在不同的镜像中构建，这些中间镜像会被抛弃，不会影响到最终镜像，因此无需费力减少中间层和文件。

- 安装构建应用程序所需的工具
- 安装或更新库依赖项
- 生成您的应用程序
- 复制最终的可执行文件

这一点在 Web 前端项目之类需要编译和转译的语言中非常有优势。例如可以使用`COPY --from=build dist/ .`拷贝`build`层的 dist 文件到 nginx 镜像中。
基底镜像包含了 node 运行时，build 层会有大量`.ts`、`.js`文件。它们将编译成最小化的静态文件，可能只有几十到几百 KB。最终拷贝到 nginx 镜像中，只有大约：
> 174MB (`node`) + 2.1 MB (`Vue`项目) = 176 MB
> 133MB (`nginx`) + 500KB (`dist`) = 133.5MB


### 启动时加载
这一点常常被人忽略，通常镜像的入口常常是 `entrypoint.sh` 或者 `docker-entrypoint.sh` 之类的脚本。可以将一些预备工作延迟到这些容器入口脚本里，例如 MySQL 的镜像在启动时，就会判断挂载目录中是否存在数据文件来决定是否初始化等等。

## DinD 构建
现在都要求容器内构建，对外部环境影响更小也更容易受控制。多阶段创建也会创建许多悬空镜像，需要及时清理。
不过现在的流水线工具都基于 `DinD`（`Docker in Docker`）。构建过程也发生在流水线对应的镜像中，一旦流水线完成，对应镜像也会被销毁，悬空镜像也会随之销毁。
代价是容器内构建无法使用依赖包的本地缓存。微服务下的项目实际编译时长并不会很长，反而依赖下载时间占构建时间 50%~70% 成为瓶颈。
