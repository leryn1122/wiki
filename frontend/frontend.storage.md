---
id: frontend.storage
tags:
- frontend
- h5
title: "\u524D\u7AEF\u5B58\u50A8"

---
# 前端存储
前端领域开发中会有需要存储数据需求，浏览器存储目前有以下几种：

| | 存储大小 | 失效时间 | 服务端交互 | 访问策略 |
| --- | --- | --- | --- | --- |
| 会话期 Cookie | 4 kb | 浏览器关闭时自动清除 | 有 | 符合同源策略可以访问 |
| 持久化 Cookie | 4 kb | 设置到期时间，到期后清楚 | 有 | 符合同源策略可以访问 |
| SessionStorage | 2.5~10 MB | 浏览器关闭时自动清除 | 否 | 符合同源策略可以访问 |
| LocalStorage | 2.5~10 MB | 永久保存，除非手动删除 | 否 | 即使符合同源策略也不可以相互访问 |
| IndexedDB | >250 MB | 永久保存，除非手动更新或删除 | 否 | 即使符合同源策略也不可以相互访问 |


## IndexedDB
浏览器存在两种数据库：WebSQL（已经废除）和 IndexedDB，前者已经废除，后者则为现在前端可以使用的数据库。

> IndexedDB 是一种底层 API，用于在客户端存储大量的结构化数据（也包括文件/二进制大型对象（blobs））。
>
> 该 API 使用索引实现对数据的高性能搜索。虽然 [Web Storage](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Storage_API) 在存储较少量的数据很有用，但对于存储更大量的结构化数据来说力不从心。
>

### 使用场景
<font style="color:rgb(25, 27, 31);">所有的场景都基于客户端需要存储大量数据的前提下：</font>

1. 数据可视化<font style="color:rgb(25, 27, 31);">等界面，大量数据，每次请求会消耗很大性能</font>
2. <font style="color:rgb(25, 27, 31);">即时聊天工具，大量消息需要存在本地</font>
3. <font style="color:rgb(25, 27, 31);">其它存储方式容量不满足时，不得已使用 IndexedDB</font>

### 特点
1. 非关系型数据库：以键值对的形式存储数据
2. 持久化存储：
3. 异步操作：DB 操作时不会锁死浏览器，用户仍然可以进行其他操作。LocalStorage 则是同步的。
4. 支持事务：
5. 同源策略：
6. 存储用量大：

