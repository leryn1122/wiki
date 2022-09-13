<a name="ETtiD"></a>
# Jenkins 安装手册
![image.png](./assets/1662020535807-e918e80c-2095-457a-97a7-ccfe593dfec0.png)<br />参考文档:

- 
- 

Jenkins 是最受欢迎的自动化构建工具, 有丰富的扩展和插件. 许多流水线工具都内置 Jenkins 来实现自动化构建.<br />在云原生中, Jenkins 迎来了升级版 Jenkins X.<br />最新的消息中 Jenkins 已经开始了 JDK 17 版本的预览, 在不远的将来会全面迁移到 JDK 17.
<a name="lCMVr"></a>
## Docker 安装
Dockerhub 上 **jenkins** 和 **jenkinsci/jenkins** 的镜像已经 deprecated 了, 改用官方 [jenkins/jenkins](https://hub.docker.com/r/jenkins/jenkins) 镜像.<br />版本

- `jenkins/jenkins:lts-jdk11`: 每周的 latest 版本 `lts-jdk11`
- `jenkins/jenkins:jdk11`: 最新版本 `jdk11`
- `jenkins/jenkins:2.319.3`: 我司裸机部署的版本 (JDK 8)
- `jenkins/jenkins:2.365-jdk11`: 当前最新版本

这里由于很多插件和版本有关, 所以注意兼容性. 我解决方案是找最大的固定版本号, 拉最新的依赖, 截止今天最新版是 `2.365`, 选择它即可.
```bash
docker stop jenkins && docker rm jenkins
docker run \
  --detach=true \
  --publish=8061:8080 \
  --publish=50000:50000 \
  --restart=always \
  --volume=/data/jenkins:/var/jenkins_home \
  --name=jenkins \
  --hostname=jenkins \
  jenkins/jenkins:2.365-jdk11
```
首次进入网页需要会自动初始化, 如果初始化比较慢, 是由于访问官网太慢.<br />下载对应版本的 [default.json](https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/dynamic-stable-2.346.1/update-center.json) 并执行如下命令生成新的 JSON 文件. 把他挂到 nginx 上公网可以访问. 可以更换源 `hudson.model.UpdateCenter.xml`. 将其中的源更换为如上 nginx 的地址. 重启 Jenkins.
```bash
wget -o default.json https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/dynamic-stable-2.346.1/update-center.json
sed -i 's/https:\/\/updates.jenkins.io\/download/http:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json
sed -i 's/https:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
mv default.json /var/lib/jenkins/updates/
```
安装需要查看密码, 密码会写在文件中, 首次启动也会打印在 docker log中.
```bash
docker exec jenkins \
  cat /var/jenkins_home/secrets/initialAdminPassword
```
安装插件, 可能由于网络问题安装失败, 失败也可以在之后手动安装. 后续安装界面提示操作即可.
<a name="OLwyH"></a>
## Helm 安装
参考文档:

- 

手动创建 namespace, pv, pvc
```bash
kubectl create ns jenkins
```
然后配置 Helm:
```bash
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm install jenkinsci/jenkins jenkins -n jenkins
```
```properties
controller.ingress.enabled=true
controller.ingress.hostName=jenkins.domain.com
persistence.enabled=true
persistence.existingClaim=jenkins-pvc
```
<a name="eqm6H"></a>
## 配置
<a name="MA9wl"></a>
### GitHub Proxy 代理
加速 GitHub clone 的代理设置
```bash
git config --global protocol.https.allow always
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"
```
