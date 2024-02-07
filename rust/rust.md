
# Rust
参考文档：

- [https://www.rust-lang.org](https://www.rust-lang.org)
- [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)
- [https://forge.rust-lang.org/infra/other-installation-methods.html](https://forge.rust-lang.org/infra/other-installation-methods.html)
- [https://cargo.budshome.com/reference/source-replacement.html](https://cargo.budshome.com/reference/source-replacement.html)

## Rust 环境安装

### 安装包安装
Rust 官方提供了最简单的下载安装方式，一键安装 Rust 环境（Rust 的非常**干净**）。打开 rust 官网会自动检测浏览器当前的系统，提供对应系统上的命令：
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Windows 安装直接下载对应的二进制安装程序 `rustup-init.exe` 即可。<br />Windows 环境下安装开始前会检测 C++ tools 环境。如果没有的话，之后的编译时会报错。我们先忽略，继续安装，后面会介绍安装 C++。
> Rust Visual C++ prerequisites
> 
> Rust requires the Microsoft C++ build tools for Visual Studio 2013 or
> later, but they don't seem to be installed.
> 
> The easiest way to acquire the build tools is by installing Microsoft
> Visual C++ Build Tools 2019 which provides just the Visual C++ build
> tools:
> 
>   [https://visualstudio.microsoft.com/visual-cpp-build-tools/](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
> 
> Please ensure the Windows 10 SDK and the English language pack components
> are included when installing the Visual C++ Build Tools.
> 
> Alternately, you can install Visual Studio 2019, Visual Studio 2017,
> Visual Studio 2015, or Visual Studio 2013 and during install select
> the "C++ tools":
> 
>   [https://visualstudio.microsoft.com/downloads/](https://visualstudio.microsoft.com/downloads/)
> 
> Install the C++ build tools before proceeding.
> 
> If you will be targeting the GNU ABI or otherwise know what you are
> doing then it is fine to continue installation without the build
> tools, but otherwise, install the C++ build tools before proceeding.
> 
> Continue? (y/N)

安装完后测试命令：
```bash
rustc --version
cargo version
```
卸载和升级也非常的方便：
```bash
# 升级
rustup update

# 卸载
rustup self uninstall
```

### 配置 C++ 环境
参考文档：

- [Visual Cpp Build Tools - MS官网下载](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
- [Visual Studio - MS官网下载](https://visualstudio.microsoft.com/downloads/)
- [msys2 - 官网](https://www.msys2.org/)

有两种模式：MVSC 模式和 gnu 模式。<br />前者按照官网的指示下载 Microsoft 的 C++ 工具即可。大约需要 8 ~ 9G 硬盘空间。后者需要安装 MinGW 或者 mysy2 然后根据一下的步骤配置切换工具链即可，切换工具链时，会自动重新下载对应的工具链。<br />如果无法使用可以配置环境变量：
```bash
cat <<\EOF >> ~/.bashrc
PATH=~/.cargo/bin:$PATH
EOF

source ~/.bashrc
```
安装 msys2 后切换 Rust 工具链模式：
```bash
rustup toolchain install stable-x86_64-pc-windows-gnu
rustup default stable-x86_64-pc-windows-gnu
```

### 更换 cargo 源
编辑用户目录下的 `~/.cargo/config` 即可。
```bash
vim ~/.cargo/config
```
```toml
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
# 指定镜像
replace-with = '镜像源名' # 如：tuna、sjtu、ustc，或者 rustcc

# 注：以下源配置一个即可，无需全部

# 中国科学技术大学
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
# >>> 或者 <<<
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

# 上海交通大学
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index/"

# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

# rustcc社区
[source.rustcc]
registry = "https://code.aliyun.com/rustcc/crates.io-index.git"
```
