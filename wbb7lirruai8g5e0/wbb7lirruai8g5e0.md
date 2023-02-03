<a name="N1uS6"></a>
# Kindling 监控
参考文档：

- [http://kindling.harmonycloud.cn/docs/overview-and-concepts/overview/](http://kindling.harmonycloud.cn/docs/overview-and-concepts/overview/)
- [https://github.com/KindlingProject/kindling](https://github.com/KindlingProject/kindling)

Kindling 监控，采用 cgo 混编，基于 eBPF 的监控工具，以 DaemonSet 的方式在 Kubernetes 中代理收集系统调用。<br />它有两种模式：

- 一种开放 Prometheus exporter 导出数据；
- 一种重型且完整的链路部署，采用 Kafka 缓冲和 ElasticSearch 存储数据；

安装条件：

- 需要 Linux 4.17 以上的内核（因为 eBPF 不能在老版本的内核上工作）
- 可能触发和 kube-proxy 的冲突，目前实测没法发生

eBPF 是一种最近开始流行的 Linux 内核原理，有些开发者用它来制作 GFW 过滤插件。
```bash
curl -O https://k8s-bpf-probes-public.oss-cn-hangzhou.aliyuncs.com/kindling-install.tar

tar -xvf kindling-install.tar && cd kindling-install

chmod u+x *.sh

sh install.sh
```
如果你的 `kindling-agent` 启动报错，查看日志很有可能基于当前系统的内核版本来重新构建 agent：
```bash
# 安装内核头文件
apt -y install linux-headers-$(uname -r)

# 重新编译镜像
bash -c "$(curl -fsSL https://k8s-bpf-probes-public.oss-cn-hangzhou.aliyuncs.com/recompile-module.sh)"

# 重新命名你的新镜像
docker tag kindlingproject/kindling-agent:latest-bymyself \
  harbor.leryn.top/infra/kindlingproject/kindling-agent:v0.6.0-$(uname -r)
docker push harbor.leryn.top/infra/kindlingproject/kindling-agent:v0.6.0-$(uname -r)
```
你可以使用一下端点来采集数据，或者使用自带的 UI 来展示：
```bash
curl -XGET http://localhost:9500/metrics
```
