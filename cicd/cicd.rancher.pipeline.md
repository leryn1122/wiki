<a name="XRnsM"></a>
# Rancher Pipeline

Rancher 内置的流水线是世界上配置最简单的流水线, 仅仅需要两步:

- 在代码仓库中配置 `.rancher-pipeline.yaml`
- 在 Rancher 中使用 OAuth2 登录当前使用的 git 仓库服务, 例如 GitLab. 然后开启流水线服务即可.

<a name="YZdp8"></a>
## 示例 .rancher-pipeline.yaml

以下是官网的一些示例:

```yaml
# Rancher官网示例配置文件
# https://docs.rancher.cn/docs/rancher2.5/pipelines/config/_index/
# 示例
stages:
  - name: Build something
    # 触发阶段的条件
    when:
      branch: master
      event: [push, pull_request]
    # 同步运行的多个步骤
    steps:
      - runScriptConfig:
          image: busybox
          shellScript: date -R
  - name: Publish my image
    steps:
      - publishImageConfig:
          dockerfilePath: ./Dockerfile
          buildContext: .
          tag: rancher/rancher:v2.0.0
          # （可选）是否推送到远端仓库
          pushRemote: true
          registry: reg.example.com
---
#
# https://github.com/rancher/pipeline-example-maven/blob/master/.rancher-pipeline.yml
stages:
  - name: Build
    steps:
      - runScriptConfig:
          image: maven:3-jdk-7
          shellScript: |-
            mvn clean install
  - name: Publish
    steps:
      - publishImageConfig:
          dockerfilePath: ./Dockerfile
          buildContext: .
          tag: example-greenhouse:${CICD_EXECUTION_SEQUENCE}
  - name: Deploy
    steps:
      - applyYamlConfig:
          path: ./deployment.yaml
---
#
# https://github.com/rancher/pipeline-example-go/blob/master/.rancher-pipeline.yml
stages:
  - name: Build
    steps:
      - runScriptConfig:
          image: golang:1.11
          shellScript: |-
            mkdir -p /go/src/github.com/rancher
            ln -s `pwd` /go/src/github.com/rancher/pipeline-example-go
            cd /go/src/github.com/rancher/pipeline-example-go
            go build -o bin/hello-server
            go test -cover
  - name: Publish
    steps:
      - publishImageConfig:
          dockerfilePath: ./Dockerfile
          buildContext: .
          tag: example-helloserver:${CICD_EXECUTION_SEQUENCE}
  - name: Deploy
    steps:
      - applyYamlConfig:
          path: ./deployment.yaml

```
