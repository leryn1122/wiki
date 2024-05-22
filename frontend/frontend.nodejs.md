---
id: frontend.nodejs
tags:
- frontend
- nodejs
title: Node.js

---


# Node.js
参考文档：

- [Node.js](https://nodejs.org/en/)
- [How To Install Node.js on Ubuntu 20.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)

Node.js （简称 node）是一个 2009 年诞生由 C++ 编写基于 Chrome v8 内核作为 JavaScript 引擎，使用了事件驱动、非阻塞式 I/O 模型，它是一个轻量级且高效的 JavaScript 引擎。它提供了 JavaScript 脱离浏览器环境运行的能力，从此 JavaScript 也可以用于服务端开发，相比于同时期的 php 性能优秀不少。node 也为大家带了前端项目的工程化、模块化。国外的程序员仍然非常青睐使用 node 开发轻量级的 Web 服务器，可以快速开发并验证业务。
node 会自带一个 npm（Node Package Manager）作为包管理器。
现在 node 的用途主要由两个：

- 用于 JavaScript 执行器，可执行 JavaScript 开发的工具链和开发环境的本地服务进程
- 轻量级 Web 后端服务


## 安装手册


### 包管理器安装（推荐）
如果你使用默认存储库安装带有 apt 的 Node.js，下载的版本会比较古老。所以请使用 NodeSource PPA 安装带有 apt 的 Node.js。PPA 拥有比官方 ubuntu 存储库更多的 Node.js 版本。
```bash
curl -sL https://deb.nodesource.com/setup_16.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt update && sudo apt install -y nodejs
```
```bash
node -v
```


### 启动与验证
新建 JavaScript 脚本 `hello_world.js` 用于验证：
```javascript
console.log('Hello World!!');
```
使用 `node` 命令运行该 JavaScript 脚本文件，终端会打印出一行`Hello World!!`
```bash
node hello_world.js
```


### 安装全局插件
这里比较建议使用 pnpm 作为包管理器，pnpm 安装依赖时不会重复安装依赖，该改用硬链接的方式指向了统一的依赖。因此它的下载速度更快，磁盘占用更小。
先用 npm 手动安装 pnpm，之后再用 pnpm 作为包管理器安装依赖。
```bash
sudo npm install -g pnpm --registry=https://registry.npm.taobao.org
```
