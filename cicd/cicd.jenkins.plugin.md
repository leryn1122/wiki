<a name="CzzK8"></a>
# Jenkins 插件
本文会列举常用插件的用法, 主要是 JCasC.
<a name="iIWlU"></a>
## Configuration as Code (JCasC)
参考文档:

- [https://plugins.jenkins.io/configuration-as-code/](https://plugins.jenkins.io/configuration-as-code/)
> `JCasC = Jenkins Configuration as Code`

它允许用户用可读的、声明式的 YAML 配置各种参数, 而不是在页面上点击按钮添加配置.<br />如果你需要通过命令行启动 Jenkins Docker 容器, 那么它会自动初始化 Jenkins 实例.

它查找 `CASC_JENKINS_CONFIG` 环境变量或者 `casc.jenkins.config` Java 参数 (逗号分隔) :

- 包含一组配置文件的文件夹的路径. 例 `/var/jenkins_home/casc_configs`
- 单个文件路径. 例 `/var/jenkins_home/casc_configs/jenkins.yaml`
- 指向网络 URL. 例 `https://acme.org/jenkins.yaml`

如果 `CASC_JENKINS_CONFIG` 指向文件夹, 将递归遍历 `.yml`, `.yaml`, `.YAML`, `.YML `后缀文件<br />默认地址是 `$JENKINS_HOME/jenkins.yaml`.


