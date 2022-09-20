<a name="Th00x"></a>
# backup.sh
注意这是 docker 版的 mysql 备份脚本:
```bash
#!/usr/bin/env bash

source /etc/profile

DB_HOST=$1
DB_USER=$2
DB_PASSWORD=$3
DB_PORT=3306

BACKUP_DIR="/data/mysql-files"
if [ ! -d $BACKUP_DIR ]; then
  mkdir -p $BACKUP_DIR
fi

LOGFILE="$BACKUP_DIR/backup.log"
KEEPDAY=14
TIME="$(date +"%Y%m%d_%H%M%S")"

echo "$(date +"%Y-%m-%d %H:%M:%S") Full backup started!" >> $LOGFILE

docker exec mysql bash -c \
  "mysql -u$DB_USER -p$DB_PASSWORD -P$DB_PORT -h$DB_HOST -NBe 'show databases;'" | \
  grep -vE "information_schema|performance_schema|sys|Warning" | sed 's/\r//p' 1>$BACKUP_DIR/db_name.txt

while read -r db ; do
  docker exec mysql bash -c \
    "mysqldump -u$DB_USER -p$DB_PASSWORD -P$DB_PORT -h$DB_HOST --flush-logs --single-transaction --databases $db " | \
    tee -a > $BACKUP_DIR/${TIME}_$db.sql
done< "$BACKUP_DIR/db_name.txt"

echo "$(date +"%Y-%m-%d %H:%M:%S") Windup expired backup!" >> $LOGFILE
find $BACKUP_DIR -name "*.sql" -mtime +$KEEPDAY -exec rm -rf {} \; >/dev/null 2>&1

echo "$(date +"%Y-%m-%d %H:%M:%S") Full backup finished!" >> $LOGFILE
exit 0
```
