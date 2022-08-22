![](https://s3.leryn.top/website/image/aws-s3.svg#crop=0&crop=0&crop=1&crop=1&height=77&id=DIrhV&originHeight=770&originWidth=2500&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=250)

<a name="EmjYU"></a>
# Amazon S3 对象存储

参考文档:

- 
- 

OSS 完全符合 Amazon S3 的规范, 所有文档参考 Amazon S3 文档即可.<br />S3 适合维护了一个扁平化的索引, 不存在传统意义上的文件夹的概念. 适合大量小文件的存储以及读多写少的场景.

<a name="61a3ec66"></a>
## 介绍

界面上配置 S3 用户, 之后会拿到两个密钥: 访问密钥(Access Key)和安全密钥(Secret Key), 需要妥善保存, 有点类似于 Oauth2 中的 ClientId 和 ClientSecret.<br />存储桶是 S3 中的对象容器, 通俗的话可以理解成文件系统的驱动器(C 盘, D 盘). 存储桶名是全局唯一的, 不能重复创建. 桶之间的对象是隔离的, 除非你明确转移桶内的资源.<br />存储桶中的对象.<br />对象用存储桶名和本身键名唯一确定. 键名可以是一个类似于文件路径的字符串

<a name="SDK"></a>
## SDK

这里着重将 S3 Java 的 SDK. 如果需要可以使用.

<a name="8f7aab1a"></a>
### Maven 依赖

```xml
<properties>
    <amazon-s3.version>1.11.699</amazon-s3.version>
</properties>

<!-- https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-s3 -->
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3</artifactId>
    <version>${amazon.s3.version}</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-dynamodb -->
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-dynamodb</artifactId>
    <version>${amazon-s3.version}</version>
</dependency>
```

<a name="58e6667b"></a>
#### 常见的 SDK

如果需要自学可以低价开通阿里云 OSS 服务, 使用阿里云的 SDK, 和 AmazonS3 基本大同小异, 大部分情况只是换个包名.

```xml
<mirrors>
    <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>

<properties>
    <aliyun-oss.version>3.13.2</aliyun-oss.version>
</properties>

<!-- https://mvnrepository.com/artifact/com.aliyun.oss/aliyun-sdk-oss -->
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>${aliyun-oss.version}</version>
</dependency>
```

**Minio SDK**

```xml
<properties>
    <minio.version>8.3.4</minio.version>
</properties>

<!-- https://mvnrepository.com/artifact/io.minio/minio -->
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>${minio.version}</version>
</dependency>
```

<a name="c8b57520"></a>
### Java 代码

创建一个客户端连接, 以下代码可以创建可以客户端连接, 客户端连接是一个需要及时关闭的对象, 但不是`Closeable`的对象. 客户端连接前需要配置连接信息.

```properties
accessKey=*****  // 前文所述 Access Key
secretKey=*****  // 前文所述 Secret Key
endpoint=http://www.sample-endpoint.com
bucketName=sample_bucket_name
```

```java
AWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);

ClientConfiguration clientConfiguration = new ClientConfiguration();
clientConfiguration.setProtocol(Protocol.HTTP);
clientConfiguration.setConnectionTimeout(10000);

AmazonS3 amazonS3 = AmazonS3ClientBuilder.standard()
  .withCredentials(new AWSStaticCredentialsProvider(credentials))
  .withClientConfiguration(clientConfiguration)
  .withEndpointConfiguration(new AwsClientBuilder.EndpointConfiguration(endpoint, "")) // 私有云中第二个参数任意填, 公有云中需要填地区
  .withPathStyleAccessEnabled(true)
  .build();
```

简单的上传下载查询:

```java
// 查询
ObjectListing objectListing = amazonS3.listObjects(bucketName);
List<S3ObjectSummary> objectSummaries = objectListing.getObjectSummaries();
List<String> objects = new ArrayList<>();
objectSummaries.stream().map(S3ObjectSummary::getKey).forEach(objects::add);

// 上传
ObjectMetadata objectMetadata = new ObjectMetadata();
// 可以上传本地文件
PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, objectKeyName, file);
// 或者使用IO流, 此时必须要告诉其元数据
PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, objectKeyName, inputStream, objectMetadata);
putObjectRequest.setMetadata(objectMetadata);
amazonS3.putObject(putObjectRequest);

// 删除
DeleteObjectRequest deleteObjectRequest = new DeleteObjectRequest(bucketName, objectKeyName);
amazonS3.deleteObject(deleteObjectRequest);

// 直接下载
// TODO

// 生成下载直链
EXPIRATION_MILLIS = 7 * 24 * 3600 * 1000L
Date expiration = new Date(System.currentTimeMillis() + EXPIRATION_MILLIS);
URL url = amazonS3.generatePresignedUrl(bucketName, objectKeyName, expiration);

amazonS3.shutdown();
```
