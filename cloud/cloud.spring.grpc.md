
几种 RPC 调用方法：

- RESTful & JSON HTTP
- gRPC HTTP2.0 Protobuf
- Thrift

## RESTful & JSON HTTP
Spring Boot 直接使用 `RestTemplate` 调用 API 接口即可。
```java
ResponseEntity<Map> responseEntity = restTemplate.getForEntity("https://docker.leryn.top/v2/_catalog", Map.class);
```
优点：

- 调用方便
- 跨语言
- 易于扩展

缺点：

- Http Header 和 JSON 冗余使得网络传输效率低
- JSON 动态类型，结构复杂

## gRPC HTTP2.0 Protobuf
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
