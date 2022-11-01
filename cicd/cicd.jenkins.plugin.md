<a name="CzzK8"></a>
# Jenkins 插件
本文会列举常用插件的用法，主要是 JCasC。之后插件的配置都会以 JCasC 的 YAML 配置为基础。如果 Jenkins 可配置化后，一切部署和调用都可以实现自动化了。
<a name="iIWlU"></a>
## JCasC - Jenkins Configuration as Code
参考文档：

- [https://plugins.jenkins.io/configuration-as-code/](https://plugins.jenkins.io/configuration-as-code/)
> `JCasC = Jenkins Configuration as Code`

它允许用户用可读的、声明式的 YAML 配置各种参数，而不是在页面上点击按钮添加配置。<br />如果你需要通过命令行启动 Jenkins Docker 容器，那么它会自动初始化 Jenkins 实例。
<a name="JORDZ"></a>
### 配置方式
它查找 `CASC_JENKINS_CONFIG` 环境变量或者 `casc.jenkins.config` Java 参数 (逗号分隔) :

- 包含一组配置文件的文件夹的路径。例 `/var/jenkins_home/casc_configs`
- 单个文件路径。例 `/var/jenkins_home/casc_configs/jenkins.yaml`
- 指向网络 URL。例 `https://acme.org/jenkins.yaml`

如果 `CASC_JENKINS_CONFIG` 指向文件夹, 将递归遍历 `.yml`, `.yaml`, `.YAML`, `.YML` 后缀文件<br />默认地址是 `$JENKINS_HOME/jenkins.yaml`。
<a name="eycHR"></a>
### 刷新配置
更新配置后，需要调用 Jenkins 重新加载配置的 RESTful API。<br />如果使用的是 Helm 的安装方式，那么 Jenkins 的 StatefulSet 下会有个名为 `config-reload` 的 SideCar 用于解决这个问题。它会自动监听 JCasC 对应的 ConfigMap 的变化，如果数据更新自动调用重新加载的接口。
<a name="XVR3G"></a>
## Kubernetes


<a name="EJELY"></a>
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
