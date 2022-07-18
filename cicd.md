<a name="BnKGA"></a>
# CI/CD
写在前面的一点废话: gitlab 有一个 group 和 subgroup 的概念, 公司通常把项目按照项目结构分配在各个 group 下, 例如官网应用就在 homepage 下 homepage-frontend 和 homepage-backend. 遗憾的是, 调研很多应用都不支持这个特色, 例如 zadig 和 drone.
<a name="MoZPj"></a>
## CI/CD 流程

![](https://cdn.mirantis.com/wp-content/uploads/2020/01/cicd.png#crop=0&crop=0&crop=1&crop=1&id=ugeMe&originHeight=902&originWidth=1600&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![](https://miao-blog-md.oss-cn-qingdao.aliyuncs.com/img/ae6987f7e22871a3e891fbbf6468096f.png#crop=0&crop=0&crop=1&crop=1&id=SITf2&originHeight=668&originWidth=890&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />实践下来, 目前 rancher 是最简单的解决方案. 可惜暂时不支持 gitee 或 gitea, 仅仅支持 gitlab 和 github.<br />helm 目前是一个比较难上手的 yaml 模板渲染工具, 使用架构师已经写好的 template 定制化公司内部的框架开发.

我司的 CI/CD 流程:

```
graph LR
  dev[开发人员]-->gitlab[GitLab提交代码];
  gitlab-->ci[Rancher Pipeline自动打包];
  ci-->docker[向Docker Registry仓库上传镜像];
  ci-->helm[Helm charts商店];
  docker-->publish[K8S API发布应用];
  helm-->publish;
```

本人阿里云的目前的架构, 多方考量目前最适合我本地的架构:

- gitee 不需要私有部署;
- drone 比 rancher, jenkins 内存开销更小, rancher 需要依托本地 k8s, jenkins 约 1G 运行内存;
- k8s 内存和硬盘开销都比较大, 即使 minikube 和 k3s 基本都在 4G 内存/20G 硬盘左右, 而且对出网 IO 有明显的影响;

```bash
graph LR
  dev[开发人员]-->git[Gitee提交代码];
  git-->ci[Drone CI Pipeline自动打包];
  ci-->docker[向Docker Registry仓库上传镜像];
  docker-->publish[Docker发布应用];
```
