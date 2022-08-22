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
