---
id: nosql.etcd
tags:
- etcd
- nosql
title: Etcd

---


# Etcd
参考文档：

- [etcd](https://etcd.io/)

Etcd 是一个 CoreOS 开源的高可用强一致性分布式键值数据库，具有一下特点：

- 接口简单：可以使用简单的 HTTP 接口读写
- 键值数据库：使用层级目录结构存储键值，逻辑上如同使用文件系统
- 监测变更：监测特定的键或目录以进行更改，并对值的更改做出反应
- 安全：支持 SSL 安全配置
- Benchmark：单实例支持每秒 1000+ 写操作
- 可靠：使用 Raft 算法，实现分布式系统的可用性和强一致性
```bash
etcd 
```
```bash
etcdctl put greeting "Hello, etcd"
etcdctl get greeting
```
