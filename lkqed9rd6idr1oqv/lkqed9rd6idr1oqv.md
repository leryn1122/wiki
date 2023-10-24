

```bash
# 第一行是 BPF 环境需要的依赖
# 第二行是 cargo generate 可能需要的依赖
sudo apt install -y \
  llvm clang linux-tools-common bpftool \
  openssl pkg-config libssl-dev gcc m4 ca-certificates make perl

cargo install bpf-linker
cargo install cargo-generate
```
生成 Rust 项目模版：
```bash
cargo generate https://github.com/aya-rs/aya-template
```
