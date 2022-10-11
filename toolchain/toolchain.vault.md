<a name="X2nG0"></a>
# Vault 安装手册

参考文档:

- [https://github.com/hashicorp/vault](https://github.com/hashicorp/vault)
- [https://hub.docker.com/_/vault](https://hub.docker.com/_/vault)
- [http://t.zoukankan.com/kirito-c-p-14327708.html](http://t.zoukankan.com/kirito-c-p-14327708.html)

```bash
docker run \
  --cap-add=IPC_LOCK \
  --detach=true \
  --env=VAULT_LOCAL_CONFIG='{"backend": {"file": {"path": "/vault/file"}}, "default_lease_ttl": "168h", "max_lease_ttl": "720h"}' \
  --publish=8200:8200 \
  --hostname=vault \
  --name=vault \
  vault server
```