<a name="ETtiD"></a>
# Jenkins 安装手册
参考文档:

- [https://www.jenkins.io/](https://www.jenkins.io/)
- [https://wiki.eryajf.net/pages/2415.html](https://wiki.eryajf.net/pages/2415.html)

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
## Helm 安装 (云环境推荐)
参考文档:

- [https://github.com/jenkinsci/helm-charts](https://github.com/jenkinsci/helm-charts)

手动创建 namespace, pv, pvc
```bash
kubectl create ns jenkins
```
然后配置 Helm:
```bash
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm install jenkins jenkinsci/jenkins -n jenkins
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
加速 GitHub clone 的代理设置:
```bash
git config --global protocol.https.allow always
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"
```
<a name="lCboJ"></a>
## Jenkins Docker
参考文档:

- [https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#run-jenkins-in-docker](https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#run-jenkins-in-docker)

如果需要容器化部署 Jenkins 同时在 Pipeline 中调用 Docker 命令等等, 请按照以上文档中的步骤构建自己的 Jenkins-docker 镜像.
```bash
docker build . -f Dockerfile -t harbor.leryn.top/infra/jenkins-docker:2.365-jdk11
```
```dockerfile
FROM jenkins/jenkins:2.365-jdk11 AS base

USER root

RUN apt-get update && apt-get install -y lsb-release \
      && \
    curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg \
      && \  
    echo "deb [arch=$(dpkg --print-architecture) \
          signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
          https://download.docker.com/linux/debian \
          $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
      && \
    apt-get update && apt-get install -y docker-ce-cli

USER jenkins

# 如果外网访问非常慢的话可以注释掉这句话手动安装插件
RUN jenkins-plugin-cli --plugins "blueocean:1.25.6 docker-workflow:1.29"
```

```bash
docker build . -f Dockerfile -t harbor.leryn.top/infra/jenkins-agent-docker:4.10-3-jdk11
```
```dockerfile
FROM jenkins/inbound-agent:4.10-3-jdk11 AS base

USER root

RUN apt-get update && apt-get install -y lsb-release curl \
      && \
    curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg \
      && \  
    echo "deb [arch=$(dpkg --print-architecture) \
          signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
          https://download.docker.com/linux/debian \
          $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
      && \
    apt-get update && apt-get install -y docker-ce-cli

USER jenkins

ENTRYPOINT [ "/usr/local/bin/jenkins-agent" ]
```
<a name="S75dd"></a>
# 访问 Jenkins
<a name="YyHxK"></a>
## Jenkins CLI
```bash
java -jar jenkins-cli.jar -s https://jenkins.leryn.top/ -auth admin:1159a750229c40a61247baaff72f75b9b5 -webSocket help
```
<a name="R0yo2"></a>
## Jenkins Crumb
Jenkins Crumb 实际上相当于其他 Web 服务中的 Token. 它用于防止 CSRF 攻击.<br />以下方式生产 Crumb:
```bash
curl -XGET http://jenkins.domain.com/crumbIssuer/api/json --user admin:admin
# 或者
curl -XGET https://jenkins.leryn.top/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb) --user admin:admin
```
```json
{"_class":"hudson.security.csrf.DefaultCrumbIssuer","crumb":"xxxxxx","crumbRequestField":"Jenkins-Crumb"}
# 或者
Jenkins-Crumb:xxxx
```
使用然后使用 Crumb + Authorization 触发流水线:
```bash
# 无参
curl -XPOST https://jenkins.domain.com/job/$PROJECT/job/$PIPELINE/build \
  -H 'Jenkins-Crumb:xxxx' \
  -u admin:admin

# 或者用表单提交参数
curl -XPOST https://jenkins.domain.com/job/$PROJECT/job/$PIPELINE/buildWithParameters \
  -H 'Jenkins-Crumb:xxxx' \
  -u admin:admin \
  -form xxxx
```
 这个请求不会返回任何 Body, 但是响应的 Header 上 `Location` 中会返回唯一的 Queue ID.
```bash
http://jenkins.leryn.top/queue/item/313/
```