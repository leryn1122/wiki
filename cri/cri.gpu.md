
# NVIDIA GPU 容器运行时
参考文档：

- [Overview - NVIDIA Container Toolkit Documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/index.html)

## NVIDIA Container
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
MIG（Multi-Instance GPU）是一种 Ampere 架构显卡上的特性。通过硬件设计，将 GPU 分为更小的子 GPU。依次建立 GI（GPU Instnace）和并在其上创建 CI（Computed Instance）。MIG-vGPU 技术性能优于基于 Time-sliced 的 vGPU 技术。
MIG 设备编号：例如 1c.3g.20gb 表示 1c = 一个计算单位，3g.20gb = 3G GPU内存和 20G内存。
