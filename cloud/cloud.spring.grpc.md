---
id: cloud.spring.grpc
tags:
- grpc
title: RPC

---


# RPC - 远程过程调用
参考文档：

- [gRPC](https://grpc.io/)
- [HTTP VS RPC / Feign VS Dubbo_rpc和feign的区别-CSDN博客](https://blog.csdn.net/cristianoxm/article/details/120567823?utm_source=miniapp_weixin)

RPC（Remote Procedure Call）就是远程过程调用，通俗来讲就是远程调用其他服务的方法。
你可以认为 HTTP 调用远端服务接口也是一种 RPC，但是这样会让 RPC 的概念变得过于宽泛了。
HTTP 本身是一种相对低效的传输协议：

- HTTP 包含请求头作为请求的元数据，但是这部分数据量非常大，而且全部采用纯文本传输。这部分请求头可能会比一些小接口响应体的本身还要大，请求头大小占比在 HTTP 中也占据了相当大一部分。
- HTTP 本身是无状态的传输协议，所以通过 Cookie、Token 等技术实现往往会进一步增加请求头的占比。
- JSON 作为一种数据的序列化方案，它包含的了大量冗余的纯文本符号：引号、冒号、括号等。尽管相比 XML 来说，它拥有更高的信息密度。

HTTP 也并不是全无优点，HTTP 本身：

- 调用方便，按照格式拼接纯文本即可发送简单的 HTTP 请求
- 跨语言，几乎任何语言都可以找到丰富的 HTTP 请求库
- 调试方便，明文传输让它更容易在网络抓包和断点上调试

RESTful & JSON HTTP 这并不是任何的 RPC，但还是会介绍它们。
几种 RPC 调用方法：

- gRPC HTTP2.0 Protobuf
- Thrift
- Dubbo


## RESTful & JSON HTTP
RESTful & JSON HTTP 这不是任何一种 RPC。放在这里提及是因为它们也是最常使用的 HTTP 调用的风格和约定，它和 RPC 不是一个位面上谈论的事物。
首先，HTTP 本身一是一种应用层网络传输协议。在业务实践中，慢慢产生了两张主流的接口开发风格和约定，一种是 RESTful（Representational State Transfer，表述性状态转移），它要求：

- 使用语义化的 HTTP Method：GET、POST、PUT、DELETE 分别对应了查询、新增、修改、删除
- 使用 URI 标识唯一种类的资源，尽量使用紧凑型的 URI 设计
- 通常返回 JSON 格式的数据

JSON HTTP 只要求通过 JSON 来发送请求体和接收响应体。


### Spring 的例子
Spring 中直接使用 `RestTemplate` 调用 API 接口即可：
```java
HttpEntity<GetFeatureRequest> httpEntity = new HttpEntity<>(request, headers);
ResponseEntity<GetFeatureResponse> responseEntity = restTemplate.postForEntity("https://api.mydomain.com/getFeature", httpEntity, GetFeatureResponse.class);
```
Spring Cloud 中提供了另一个声明式的 HTTP 客户端 Feign：
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
SpringBoot 的 `application.yml` 里配置 Feign：
```java
@FeignClient(name = "user-service")
public interface UserServiceFeign {
    
    @RequestMapping(method = RequestMethod.GET, value = "/user/{id}")
    User getUserById(@PathVariable("id") Long id);
}

@Service
public class OrderService {
    
    @Autowired
    private UserServiceFeign userServiceFeign;
    
    public void processOrder(Long userId) {
        User user = userServiceFeign.getUserById(userId);
        // ...
    }
}
```


## gRPC HTTP/2.0 Protobuf
gRPC（Google RPC）是 Google 开源的一种 RPC 协议，它基于 HTTP/2.0 封装上层。它是跨语言的，因为 Google 提供了主流语言的 Protobuf 的序列化库。
它将数据序列化为二进制数据，虽然对于人类不可读，但是具有更高的传输效率，并且专门优化了 Protobuf 的序列化和反序列化算法。
使用 Protobuf 的话需要根据 `.proto` 文件在编译或打包阶段预先生成对应的 Stub 文件。
对于不对外的服务接口，例如集群内部的服务间调用，完全可以使用
各种语言的 gRPC 调用需要根据官方文档来编写。


## Thrift
这是一个由 Facebook 提出的 RPC 框架，Hive 使用了这种技术。


# gRPC


## Protobuf
所有 gRPC 项目都必须编写 Protobuf 文件，再 Protobuf complier 生成对应语言的代码。另一方面，Protobuf 文件本身提供了接口的自述规范，对接的开发只需要根据同一份 Protobuf 生成自己开发语言代码即可。
```protobuf
service RouteGuide {
   rpc GetFeature(Point) returns (Feature) {
   }
}

message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```


## Python
参考文档：

- [Basics tutorial](https://grpc.io/docs/languages/python/basics/)
```bash
pipenv install grpcio-tools

pipenv run python3 -m grpc_tools.protoc -I../../protos \
  --python_out=. \
  --pyi_out=. \
  --grpc_python_out=. \
  ../proto/*.proto
```


### 服务端
注册一个 RouteGuideServicer 的 gRPC Service：
```python
class RouteGuideServicer(route_guide_pb2_grpc.RouteGuideServicer):
    def GetFeature(self, request, context):
    feature = get_feature(self.db, request)
    # TODO ...
    
        return feature
```
启动一个 50051 端口上的 gRPC 服务端。
```python
def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    route_guide_pb2_grpc.add_RouteGuideServicer_to_server(RouteGuideServicer(), server)
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```


## Golang
参考文档：

- [Quick start](https://grpc.io/docs/languages/go/quickstart/)
```bash
apt install -y protobuf-compiler

go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2

export PATH="$PATH:$(go env GOPATH)/bin"

protoc \
  --go_out=. \
  --go_opt=paths=source_relative \
  --go-grpc_out=. \
  --go-grpc_opt=paths=source_relative \
  ../proto/*.proto
```


### 服务端
```go
listener, err := net.Listen("tcp", "[::]:50051")

if err != nil {
  log.Fatalf("failed to listen: %v", err)
}

var opts []grpc.ServerOption
...
grpcServer := grpc.NewServer(opts...)
pb.RegisterRouteGuideServer(grpcServer, newServer())
grpcServer.Serve(listener)
```
```go
func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
    // TODO ...
    return &pb.Feature{Location: point}, nil
}
```


### 客户端
```go
var opts []grpc.DialOption
...
conn, err := grpc.Dial(*serverAddr, opts...)

if err != nil {
  ...
}
defer conn.Close()


client := pb.NewRouteGuideClient(conn)

feature, err := client.GetFeature(context.Background(), &pb.Point{409146138, -746188906})
if err != nil {
  ...
}
```


## Java
```xml
<build>
  <extensions>
    <extension>
      <groupId>kr.motd.maven</groupId>
      <artifactId>os-maven-plugin</artifactId>
      <version>1.6.1</version>
    </extension>
  </extensions>
  
  <plugins>
    <plugin>
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.6.1</version>
      <configuration>
        <protocArtifact>com.google.protobuf:protoc:3.6.1:exe:${os.detected.classifier}</protocArtifact>
        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.14.0:exe:${os.detected.classifier}</pluginArtifact>
        <pluginId>grpc-java</pluginId>
        <protoSourceRoot>${project.basedir}/src/main/proto</protoSourceRoot>
        <outputDirectory>${project.basedir}/src/main/java</outputDirectory>
        <clearOutputDirectory>false</clearOutputDirectory>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>compile</goal>
            <goal>test-compile</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```
