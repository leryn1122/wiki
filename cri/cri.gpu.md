---
id: cri.gpu
tags:
- gpu
- k8s
- kubernetes
- nvidia
title: "NVIDIA GPU \u5BB9\u5668\u8FD0\u884C\u65F6"

---
# NVIDIA GPU 容器运行时
参考文档：

+ [Overview — NVIDIA Container Toolkit 1.14.5 documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/index.html)
+ [NVIDIA/k8s-device-plugin源码分析-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1102413)
+ [Kubernetes教程(二二)---在 K8S 中创建 Pod 是如何使用到 GPU 的：device plugin&nvidia-container-toolkit 源码分析](https://www.lixueduan.com/posts/kubernetes/22-pod-use-gpu-in-k8s-analyze/)

## GPU on K8S
在 K8S 上使用 GPU 流程上必须要做到两件事，对应了两个组件：

+ Device-plugin：K8S 如何感知到 GPU
+ Nvidia-container-toolkit：GPU 如何分配给 Pod

Device-plugin 调用了 K8S Plugin Registry API，向 Kubelet 注册了 GPU 设备。注册成功后，节点上就知道有多少 GPU 卡了。

Pod 申请 GPU 后，device-plugin 会向容器中添加一个 `NVIDIA_VISIBLE_DEVICES` 的环境变量，对应了 GPU 卡的 UUID。



// TODO



### NVIDIA Container 安装步骤
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
    sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

配置 Docker / Containerd / CRI-O 运行时，`nvidia-ctk` 会自动修改对应运行时的配置文件：

```bash
# Docker
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Containerd
sudo nvidia-ctk runtime configure --runtime=containerd
sudo systemctl restart containerd

# 
sudo nvidia-ctk runtime configure --runtime=crio
sudo systemctl restart crio
```

查看`nvidia-smi`：

```bash
sudo nvidia-smi

# 或者
sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

## MIG
MIG（Multi-Instance GPU）原来是一种 Ampere 架构显卡上的特性，现在在所有高端 GPU 卡上都可以使用 MIG 技术了，包括 A100/A800/H800 等等。

Nvidia 公司内置的 GPU 虚拟化技术，不需要使用任何额外的 License 或技术即可开启。通过硬件设计将 GPU 分为更小的子 GPU。依次建立 GI（GPU Instnace）和并在其上创建 CI（Computed Instance）。MIG-vGPU 技术性能优于基于 Time-sliced 的 vGPU 技术。通常一个 8 卡 * 80G 显存的 GPU 服务器，每个 8 GPU 卡可以至多切分成 7 个 10G 的 MIG，也就是说一台服务器可以切分为 56 个 MIG。它可以为那些 GPU 用量较小的任务/业务来跟精准的分配 GPU，例如只需要较小的显存，不需要占用整卡。

MIG 设备编号：例如 1c.3g.20gb 表示 1c = 一个计算单位，3g.20gb = 3G GPU内存和 20G内存。





