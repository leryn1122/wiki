<a name="EnAAZ"></a>
# Jenkins 扩展共享库
参考文档:

- [https://www.jenkins.io/zh/doc/book/pipeline/shared-libraries/](https://www.jenkins.io/zh/doc/book/pipeline/shared-libraries/)
- [https://www.jenkins.io/doc/pipeline/steps/](https://www.jenkins.io/doc/pipeline/steps/)
- [https://www.jianshu.com/p/a248dc7e80ea](https://www.jianshu.com/p/a248dc7e80ea)
<a name="V0UE5"></a>
## 安装扩展库
<a name="CmCoy"></a>
### 创建仓库
创建一个 Groovy 的 git 项目, 暂时叫 `jekins-shared-lib`. 目录结构是:
```
(root)
+- src                     # Groovy source files
|   +- org
|       +- foo
|           +- Bar.groovy  # for org.foo.Bar class
+- vars
|   +- foo.groovy          # for global 'foo' variable
|   +- foo.txt             # help for 'foo' variable
+- resources               # resource files (external libraries only)
|   +- org
|       +- foo
|           +- bar.json    # static helper data for org.foo.Bar
```
> src 目录应该看起来像标准的 Java 源目录结构。当执行流水线时，该目录被添加到类路径下。
> vars 目录定义可从流水线访问的全局变量的脚本。 每个 *.groovy 文件的基名应该是一个 Groovy (~ Java) 标识符, 通常是 camelCased。 匹配 *.txt, 如果存在, 可以包含文档, 通过系统的配置标记格式化从处理 (所以可能是 HTML, Markdown 等，虽然 txt 扩展是必需的)。
> 这些目录中的 Groovy 源文件 在脚本化流水线中的 “CPS transformation” 一样。
> resources 目录允许从外部库中使用 libraryResource 步骤来加载有关的非 Groovy 文件。 目前，内部库不支持该特性。
> 根目录下的其他目录被保留下来以便于将来的增强。

<a name="xne8i"></a>
### 使用库
在 Jenkins 配置全局共享库 (文件夹共享库也是可以的) :
> Manage Jenkins » Configure System » Global Pipeline Libraries
> aka https://<jenkins.url>/manage/configure

标记为 Load implicitly 的共享库允许流水线立即使用任何此库定义的类或全局变量. 为了访问其他共享库, Jenkinsfile 需要使用 `@Library` 注解, 指定库的名字:
```groovy
@Library('jekins-shared-lib') _
/* Using a version specifier, such as branch, tag, etc */
@Library('jekins-shared-lib@1.0') _
/* Accessing multiple libraries with one statement */
@Library(['jekins-shared-lib', 'otherlib@abc1234']) _
```
<a name="rJPaC"></a>
## 创建项目

