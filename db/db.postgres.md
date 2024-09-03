---
id: db.postgres
tags:
- db
- pg
title: "Postgres \u5B89\u88C5\u624B\u518C"

---
# Postgres 安装手册


## Docker 安装


```bash
docker run \
  --detach=true \
  --name=postgres \
  --hostname=postgres \
  --env=POSTGRES_PASSWORD=oHjkz821MkhfAd33 \
  --env=PGDATA=/var/lib/postgresql/data/pgdata \
  --publish=15432:5432 \
  --volume=/data/postgres:/var/lib/postgresql/data \
  postgres:15.1-alpine
```



用连接工具连接可能会报错:



> FATAL: no pg_hba.conf entry for host "183.195.62.49", user "postgres", database "postgres", no encryption
>



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

