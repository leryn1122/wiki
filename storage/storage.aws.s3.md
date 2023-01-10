<a name="EaClw"></a>
# Amazon S3 对象存储
参考文档：

- [https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/Welcome.html](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/Welcome.html)
- [https://help.aliyun.com/document_detail/32007.html](https://help.aliyun.com/document_detail/32007.html)

OSS 完全符合 Amazon S3 的规范，所有文档参考 Amazon S3 文档即可。<br />S3 适合维护了一个扁平化的索引，不存在传统意义上的文件夹的概念。适合大量小文件的存储以及读多写少的场景。
<a name="61a3ec66"></a>
## 介绍
界面上配置 S3 用户，之后会拿到两个密钥：访问密钥（Access Key）和安全密钥（Secret Key），需要妥善保存，有点类似于 Oauth2 中的 ClientId 和 ClientSecret。<br />存储桶是 S3 中的对象容器，通俗的话可以理解成文件系统的驱动器（C 盘，D 盘）。存储桶名是全局唯一的，不能重复创建。桶之间的对象是隔离的，除非你明确转移桶内的资源存储桶中的对象。<br />对象用存储桶名和本身键名唯一确定。键名可以是一个类似于文件路径的字符串。
<a name="J1tNW"></a>
## FUSE
参考文档：

- [https://github.com/s3fs-fuse/s3fs-fuse](https://github.com/s3fs-fuse/s3fs-fuse)

介绍 SDK 前，我们先来用 FUSE 的命令行感受一下，过程会像 nfsmount 一样，先安装客户端：
```bash
# Debain/Ubuntu
apt install s3fs

# RHEL/CentOS
yum install -y epel-release
yum install -y s3fs-fuse
```
然后执行命令挂载存储桶：
```bash
# 必须是600权限
echo ACCESS_KEY_ID:SECRET_ACCESS_KEY > /etc/passwd-s3fs
chmod 600 /etc/passwd-s3fs

mkdir /path/to/mount

# 如果是 AWS S3
s3fs mybucket /path/to/mount -o passwd_file=/etc/passwd-s3fs

# 如果是非 AWS 实现需要加 URL 和 use_path_request_style
s3fs mybucket /path/to/mount \
  -o passwd_file=/etc/passwd-s3fs \
  -o url=https://oss.domain.com/ \
  -o use_path_request_style
```
<a name="SDK"></a>
## SDK
这里着重将 S3 Java 的 SDK。如果需要可以使用。
<a name="8f7aab1a"></a>
### Maven 依赖
`1.12.261` 及以下版本有安全性漏洞
```xml
<properties>
    <amazon-s3.version>1.12.290</amazon-s3.version>
</properties>

<!-- https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-s3 -->
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3</artifactId>
    <version>${amazon-s3.version}</version>
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
如果需要自学可以低价开通阿里云 OSS 服务，使用阿里云的 SDK，和 AmazonS3 基本大同小异，大部分情况只是换个包名和类名。
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
创建一个客户端连接，以下代码可以创建可以客户端连接，客户端连接是一个需要及时关闭的对象，但不是`Closeable`的对象。客户端连接前需要配置连接信息。
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
简单的上传下载查询：
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
<a name="CUiAn"></a>
## Kubernetes CSI
需要安装：

- Kubernetes 1.16+ 以上版本；
- s3fs FUSE；
- 允许特权容器；
- Docker daemon 开启 Systemd flag `MountFlags=shared`；
```bash
sudo apt install s3fs

# 检查是否是 MountFlags=shared
sudo systemctl show --property=MountFlags docker.service

# 如果结果不为空, 请在docker.service中的Service下添加, 并重启docker daemon
[Service]
MountFlags=shared

sudo systemctl daemon-reload
sudo systemctl restart docker.service
```
按照 [https://github.com/majst01/csi-driver-s3/tree/master/deploy/kubernetes](https://github.com/majst01/csi-driver-s3/tree/master/deploy/kubernetes) 这个项目目录下安装所有的 YAML，需要按照需求来创建 Secret。
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-driver-s3-secret
  namespace: kube-system
stringData:
  accessKeyID: <YOUR_ACCESS_KEY_ID>
  secretAccessKey: <YOUR_SECRET_ACCES_KEY>
  # For AWS set it to "https://s3.<region>.amazonaws.com"
  endpoint: https://s3.eu-central-1.amazonaws.com
  # If not on S3, set it to ""
  region: <S3_REGION>
```
然后按照自己的需求创建 PVC，StorageClass 指定为 `csi-driver-s3`，它会自动为你创建对应的对象存储桶和 PV，并绑定PV。
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: csi-driver-s3-pvc
  namespace: default
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: csi-driver-s3
```
<a name="XaUHo"></a>
### 注意事项

- 如果删除 PVC，那么数据桶内的资源会保存。但新建同名 PVC 时，会重新创建新的数据桶。
- 数据桶创建时无法修改是否开启版本控制等选项。
- 数据桶创建文件默认为 `application/octet-stream`，有些 S3 服务中可能会与预期的行为不一致。


