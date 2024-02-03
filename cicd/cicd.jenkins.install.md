
# Jenkins 安装手册
参考文档:

- [https://www.jenkins.io/](https://www.jenkins.io/)
- [https://wiki.eryajf.net/pages/2415.html](https://wiki.eryajf.net/pages/2415.html)

Jenkins 是最受欢迎的自动化构建工具, 有丰富的扩展和插件. 许多流水线工具都内置 Jenkins 来实现自动化构建.<br />在云原生中, Jenkins 迎来了升级版 Jenkins X.<br />最新的消息中 Jenkins 已经开始了 JDK 17 版本的预览, 在不远的将来会全面迁移到 JDK 17.

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

## 配置

### GitHub Proxy 代理
加速 GitHub clone 的代理设置:
```bash
git config --global protocol.https.allow always
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"
```

## Jenkins Docker
参考文档:

- [https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#run-jenkins-in-docker](https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#run-jenkins-in-docker)

如果需要容器化部署 Jenkins 同时在 Pipeline 中调用 Docker 命令等等, 请按照以上文档中的步骤构建自己的 Jenkins-docker 镜像.
```bash
docker build . -f Dockerfile -t harbor.mydomain.com/infra/jenkins-docker:2.365-jdk11
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

# 访问 Jenkins

## Jenkins CLI
```bash
java -jar jenkins-cli.jar -s https://jenkins.mydomain.com/ -auth admin:1159a750229c40a61247baaff72f75b9b5 -webSocket help
```

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
http://jenkins.mydomain.com/queue/item/313/
```

# Jenkins 插件
本文会列举常用插件的用法，主要是 JCasC。之后插件的配置都会以 JCasC 的 YAML 配置为基础。如果 Jenkins 可配置化后，一切部署和调用都可以实现自动化了。

## JCasC - Jenkins Configuration as Code
参考文档：

- [https://plugins.jenkins.io/configuration-as-code/](https://plugins.jenkins.io/configuration-as-code/)
> `JCasC = Jenkins Configuration as Code`

它允许用户用可读的、声明式的 YAML 配置各种参数，而不是在页面上点击按钮添加配置。<br />如果你需要通过命令行启动 Jenkins Docker 容器，那么它会自动初始化 Jenkins 实例。

### 配置方式
它查找 `CASC_JENKINS_CONFIG` 环境变量或者 `casc.jenkins.config` Java 参数 (逗号分隔) :

- 包含一组配置文件的文件夹的路径。例 `/var/jenkins_home/casc_configs`
- 单个文件路径。例 `/var/jenkins_home/casc_configs/jenkins.yaml`
- 指向网络 URL。例 `https://acme.org/jenkins.yaml`

如果 `CASC_JENKINS_CONFIG` 指向文件夹, 将递归遍历 `.yml`, `.yaml`, `.YAML`, `.YML` 后缀文件<br />默认地址是 `$JENKINS_HOME/jenkins.yaml`。

### 刷新配置
更新配置后，需要调用 Jenkins 重新加载配置的 RESTful API。<br />如果使用的是 Helm 的安装方式，那么 Jenkins 的 StatefulSet 下会有个名为 `config-reload` 的 SideCar 用于解决这个问题。它会自动监听 JCasC 对应的 ConfigMap 的变化，如果数据更新自动调用重新加载的接口。

## Kubernetes



## Simple Themes
下载后 [Simple Theme](https://wiki.jenkins-ci.org/display/JENKINS/Simple+Theme+Plugin) 插件后，

- 点开 **[Manage Jenkins]** => **[Configure System]**
- 将上面的 CSS URL 填入框内保存即可

推荐使用以下 Github Star 最高的皮肤：

- [https://github.com/afonsof/jenkins-material-theme](https://github.com/afonsof/jenkins-material-theme)
- [https://github.com/jenkins-contrib-themes/jenkins-neo-theme](https://github.com/jenkins-contrib-themes/jenkins-neo-theme)
- [https://github.com/djonsson/jenkins-atlassian-theme](https://github.com/djonsson/jenkins-atlassian-theme)

**afonsof/jenkins-material-theme**<br />[![](./../assets/1667308184771-e0b162d8-593e-4253-ac51-2844b5f7193b.png)

- 上图中挑个色号填入下文链接中的 `{{your-color-name}}` ，保存到上文配置处。
> https://cdn.rawgit.com/afonsof/jenkins-material-theme/gh-pages/dist/material-{{your-color-name}}.css

```yaml
unclassified:
  simple-theme-plugin:
    elements:
    - cssUrl:
        url: "https://cdn.jsdelivr.net/gh/afonsof/jenkins-material-theme@gh-pages/dist/material-amber.css"
```
