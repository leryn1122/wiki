---
id: network.ebpf
tags:
- ebpf
- network
title: "eBPF \u5F00\u53D1"

---


# eBPF 开发
本节指导利用 Rust 的 Aya 库来构建一个 eBPF 程序，类似的库还有 redpbf 等。
首先需要一个 Rust 的 nightly 环境：
安装必要的依赖包：
```bash
# 第一行是 BPF 环境需要的依赖
# 第二行是 cargo generate 可能需要的依赖
sudo apt install -y \
  llvm clang linux-tools-common linux-tools-generic linux-cloud-tools-generic \
  openssl pkg-config libssl-dev gcc m4 ca-certificates make perl

cargo install bpf-linker
cargo install cargo-generate
```
生成 Rust 项目模版：
```bash
cargo generate https://github.com/aya-rs/aya-template
```
编译代码：
```bash
cargo xtask build-ebpf
```
运行 eBPF 代码，这将直接加载 eBPF 程序到内核中。
```bash
RUST_LOG=info cargo xtask run -- --iface eth0

# 检查运行情况
sudo bpftool prog list

# 可以看到网卡里有一个 xdp 程序在运行
ip link show
```
