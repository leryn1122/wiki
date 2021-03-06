<a name="ETtiD"></a>
# Jenkins
![](https://www.jenkins.io/images/logos/needs-you/Jenkins_Needs_You-transparent.png#crop=0&crop=0&crop=1&crop=1&height=440&id=eFluQ&originHeight=1226&originWidth=866&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=311)<br />参考文档:

- [Jenkins - 官网](https://www.jenkins.io/)

Jenkins 是最受欢迎的自动化构建工具, 有丰富的扩展和插件. 许多流水线工具都内置 Jenkins 来实现自动化构建.<br />在云原生中, Jenkins 迎来了升级版 Jenkins X.

<a name="lCMVr"></a>
## Docker

Dockerhub 上 Jenkins 的官方镜像已经 deprecated 了, 改用 [jenkins/jenkins](https://hub.docker.com/r/jenkins/jenkins) 镜像.

```bash
docker run \
  --detach=true \
  --publish=8086:8080 \
  --restart=always \
  --volume=/data/jenkins:/var/jenkins_home \
  --name=jenkins \
  --hostname=jenkins \
  jenkins/jenkins:lts
```

首次安装需要查看密码, 密码会写在文件中.

```bash
docker exec jenkins \
       cat /var/jenkins_home/secrets/initialAdminPassword
```

安装插件, 可能由于网络问题安装失败, 失败也可以在之后手动安装. 后续安装界面提示操作即可.
<a name="cezoN"></a>
## Jenkins 配置

Jenkins 安装是很方便的, 复杂的是配置.

- 配置 Maven<br />Manage Jenkins ==> Global Tool Configuration 安装 Maven, JDK, Git 配置
- 配置 SSH<br />需要提前安装 SSH 插件


```groovy
pipeline{
    agent any
    environment{
        currentDate = sh(returnStdout: true, script: 'date +%Y-%m-%d_%H-%M-%S').trim()
    }
    stages{
        stage("checkout custormers vars"){
            steps{
                script{
                    if (env.gitlabSourceRepoHomepage){
                        repoUrl = "${env.gitlabSourceRepoHomepage}"
                    }else{
                        repoUrl = "${env.REPO_URL}"
                    }
                    if (env.gitlabBranch){
                        repoBranch = "${env.gitlabBranch}"
                    }else{
                        repoBranch = "${env.REPO_BRANCH}"
                    }
                    if (!env.HARBOR_IMAGE_NAME){
                        env.HARBOR_IMAGE_NAME = repoUrl.split('/')[-1]
                    }
                    if (env.gitlabUserUsername){
                        userName = "${env.gitlabUserUsername}"
                    }else{
                        userName = "${env.BUILD_USER}"
                    }
                }
            }
        }
        stage("Pull Souce Code"){
            steps{
                checkout([$class: "GitSCM", branches: [[name: "${repoBranch}"]], extensions: [], userRemoteConfigs: [[credentialsId: "************GitLab ClientSercet************", url: "${repoUrl}.git"]]])
            }
        }
        stage("Build and Push image"){
            steps{
                script{
                    docker.withRegistry("https://harbor.leryn.top/","************HarborToken************"){
                        def image = docker.build("${env.HARBOR_PROJECT_NAME}/${env.HARBOR_IMAGE_NAME}:${repoBranch}_${env.BUILD_NUMBER}_${env.currentDate}","-f ${env.DOCKERFILE_PATH} .")
                        image.push()
                    }
                }
            }
        }
    }
    post{
        success{
            dingtalk(
                robot: '************钉钉机器人Token************',
                type: 'MARKDOWN',
                title: "Jenkins Job Notification",
                text: [
                    "# <font color=#008000>[${env.HARBOR_IMAGE_NAME}](${repoUrl})</font>",
                    '---',
                    "- 任务ID: [${env.BUILD_NUMBER}](${env.BUILD_URL})",
                    "- 结果: **<font color=#008000>Succeeded</font>**",
                    "- 执行人: ${userName}",
                    "- 分支: ${repoBranch}",
                    "- 耗时: ${currentBuild.durationString}",
                    "- 镜像Tag: ${repoBranch}_${env.BUILD_NUMBER}_${env.currentDate}"
                ]
            )
        }
        failure{
            dingtalk(
                robot: '************钉钉机器人Token************',
                type: 'MARKDOWN',
                title: "Jenkins Job Notification",
                text: [
                    "# <font color=#008000>[${env.HARBOR_IMAGE_NAME}](${repoUrl})</font>",
                    '---',
                    "- 任务ID: [${env.BUILD_NUMBER}](${env.BUILD_URL})",
                    "- 结果: **<font color=#FF0000>Failed</font>**",
                    "- 执行人: ${userName}",
                    "- Gitlab链接: [Link](${repoUrl})",
                    "- 分支: ${repoBranch}",
                    "- 耗时: ${currentBuild.durationString}"
                ],
                at: [
                    '************钉钉注册手机号************'
                ]
            )
        }
    }
}
```
