
<a name="o8ndc"></a>
# Lua 脚本
参考文档:

- 
<a name="oJ09O"></a>
## 源码编译安装

```bash
curl -R -O http://www.lua.org/ftp/lua-5.4.4.tar.gz
tar zxf lua-5.4.4.tar.gz
cd lua-5.4.4
make all test

cd src
ln -s $PWD/lua /usr/local/bin/lua
ln -s $PWD/luac /usr/local/bin/luac 
```

或者直接下载二进制

```bash
wget https://altushost-swe.dl.sourceforge.net/project/luabinaries/5.4.2/Linux%20Libraries/lua-5.4.2_Linux415_64_lib.tar.gz
```

编译完后直接启动 lua：

```bash
./lua   
Lua 5.4.4  Copyright (C) 1994-2022 Lua.org, PUC-Rio
> print("hello, world")
hello, world
>
```
