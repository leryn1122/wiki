<a name="eDKM2"></a>
# Postgres 安装手册

<a name="wXab1"></a>
## Docker 安装

```bash
docker run \
  --detach=true \
  --name=postgres \
  --hostname=postgres \
  --env=POSTGRES_PASSWORD=oHjkz821MkhfAd33 \
  --env=PGDATA=/var/lib/postgresql/data/pgdata \
  --publish=15432:5432 \
  --volume=/conf/postgres:/etc/postgresql/ \
  --volume=/data/postgres:/var/lib/postgresql/data \
  postgres:14.1 \
    -c 'config_file=/etc/postgresql/postgresql.conf'
```

用连接工具连接可能会报错:

> FATAL: no pg_hba.conf entry for host "183.195.62.49", user "postgres", database "postgres", no encryption


```bash
vim /data/postgres/pgdata/pg_hba.conf
```

在文末加入一行, 再重启数据库:

```bash
echo 'host   all              all             0.0.0.0/0               md5' >> pg_hba.conf
```

创建数据库:

```sql
create database <Database>;
create user <Username>;
alter role <Username> with password '<Password>';
grant all privileges on database <Database> to user <Userbname>;
```
