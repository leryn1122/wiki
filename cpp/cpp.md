
# C++

## C++ 环境安装

### Linux 安装 Clang
```bash
sudo apt update
sudo apt install -y clang
```

### Windows MinGW64 安装 Clang
先用安装包安装 msys64，再安装 Clang：
```bash
echo "Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686"   >> msys64/etc/pacman.d/mirrorlist.mingw32
echo "Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64" >> msys64/etc/pacman.d/mirrorlist.mingw64
echo "Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64" >> msys64/etc/pacman.d/mirrorlist.msys
```
```bash
pacman -Sy && pacman -Su && pacman -S \
  mingw64/mingw-w64-x86_64-make \
  mingw64/mingw-w64-x86_64-gdb \
  mingw64/mingw-w64-x86_64-clang
```

