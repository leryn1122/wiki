---
id: toolchain.vault
tags:
- vault
title: Vault

---
# Vault 安装手册


参考文档:

+ [GitHub - hashicorp/vault: A tool for secrets management, encryption as a service, and privileged access management](https://github.com/hashicorp/vault)
+ [https://hub.docker.com/_/vault](https://hub.docker.com/_/vault)
+ [secrets 管理工具 Vault 的介绍、安装及使用 - 走看看](http://t.zoukankan.com/kirito-c-p-14327708.html)



```bash
docker run \
  --cap-add=IPC_LOCK \
  --detach=true \
  --env=VAULT_LOCAL_CONFIG='{"backend": {"file": {"path": "/vault/file"}}, "default_lease_ttl": "168h", "max_lease_ttl": "720h"}' \
  --publish=8200:8200 \
  --publish=8201:8201 \
  --hostname=vault \
  --name=vault \
  --volume=/data/vault/data:/vault/file \
  hashicorp/vault:1.12.2 server -dev

docker run \
  --cap-add=IPC_LOCK \
  --detach=true \
  --publish=8200:8200 \
  --hostname=vault \
  --name=vault \
  --volume=/data/vault/config:/vault/config \
  --volume=/data/vault/data:/vault/file \
  hashicorp/vault:1.12.2 server -config /vault/config/config.hcl
```

