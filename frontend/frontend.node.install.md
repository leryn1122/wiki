
# Node.js 安装手册
参考文档：

- [Node.js - 官网](https://nodejs.org/en/)
- [如何在 Ubuntu 20.04 上安装 Node.js - DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)

## 包管理器安装（推荐）

### 安装步骤
如果你使用默认存储库安装带有 apt 的 Node.js, 下载的版本会比较古老。所以请使用 NodeSource PPA 安装带有 apt 的 Node.js。PPA 拥有比官方 ubuntu 存储库更多的 Node.js 版本。
```bash
curl -sL https://deb.nodesource.com/setup_16.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt update
sudo apt install -y nodejs
```
```bash
# nodejs包包含node二进制和npm
node -v
```

## 二进制安装（不推荐）

### 前置准备
部署准备，安装前需要准备如下材料。

- Node.js 安装包: `node-v*-linux-x64.tar.xz`：
```bash
wget https://nodejs.org/dist/v18.13.0/node-v16.13.0-linux-x64.tar.xz
```

### 安装步骤
配置 Node.js 环境变量，并使其生效：
```bash
touch /etc/profile.d/env-nodejs.sh
```
```bash
#!/usr/bin/env bash
# Set Node.js environment.
export NODE_HOME=/usr/local/lib/nodejs/node
export PATH=${NODE_HOME}/bin:${PATH}
```
解压安装包到指定路径，注意权限问题，权限过低可能导致`npm install -g`失败：
```bash
VERSION=v16.13.0
DISTRO=linux-x64
sudo mkdir -p /usr/local/lib/nodejs
sudo tar -xf node-$VERSION-$DISTRO.tar.xz -C /usr/local/lib/nodejs
cd /usr/local/lib/nodejs
sudo ln -s ./node-$VERSION-$DISTRO node
```
验证 Node.js 和 npm 是否安装成功，使用命令查看版本，如果可以正确显示版本信息则安装成功：
```bash
node -v
npm -v
```
```
# node -v  ==>显示Node.js的版本信息
v16.13.0
# npm -v   ==>显示npm的版本信息
8.1.0
```

### 启动与验证
新建 JavaScript 脚本`hello_world.js`用于验证。
```bash
vim hello_world.js
```
```javascript
// 打印Hello World
console.log('Hello World!!');
```
使用`node`命令运行该 JavaScript 脚本文件，终端会打印出一行`Hello World!!`
```bash
node hello_world.js
```

## 安装全局插件
这里比较建议使用 pnpm 作为包管理器，pnpm 安装依赖时不会重复安装依赖，该改用硬链接的方式指向了统一的依赖。因此它的下载速度更快，磁盘占用更小。
先用 npm 手动安装 pnpm，之后在用 pnpm 安装所有的全局依赖。
```bash
# 包管理 && 构建工具
sudo npm install -g pnpm --registry=https://registry.npm.taobao.org
sudo pnpm install -g \
  typescript \
  vue @vue/cli vite \
  angular @angular/cli
```
如果使用了`npm`或`cnpm`作为包管理器，查看全局组件：
```bash
npm list -g
```
```
/usr/local/lib/nodejs/node-v16.13.0-linux-x64/lib
├── @angular/cli@13.0.4
├── @vue/cli@4.5.15
├── angular@1.8.2
├── cnpm@7.1.0
├── corepack@0.10.0
├── npm@8.2.0
├── pnpm@6.23.6
├── typescript@4.5.2
├── vite@2.7.1
└── vue@2.6.14
```
如果使用了`pnpm`作为包管理器，查看全局组件：
```bash
pnpm list -g
```
```
Legend: production dependency, optional only, dev only
/usr/pnpm-global/5
dependencies:
@angular/cli 13.0.4
@vue/cli 4.5.15
angular 1.8.2
typescript 4.5.2
vite 2.7.1
vue 2.6.14
```
