community version 은 cbbackupmgr 를 사용할 수가 없다.

cbbackup 과 cbrestore 를 사용해야 한다.

## Back up all nodes and all buckets

```bash
cd /opt/couchbase/bin/
./cbbackup http://127.0.0.1:8091 /yourfolder/couchbase/backups -u id-p pwd
```

## Backup shell script

```bash
su - root

cd /yourfolder/couchbase
vi backup.sh

#!/bin/bash

TIMESTAMP=`date +%Y\%m\%d\%H\%M`
OUTPUT=/yourfolder/couchbase/backups/db-backup-$TIMESTAMP;
/opt/couchbase/bin/cbbackup http://127.0.0.1:8091 $OUTPUT -u admin -p Ssgcloud#21;

echo "bakcup complete $TIMESTAMP" 
exit 0
```

```bash
# 실행 권한 부여
chmod +x backup.sh
# 등록된 잡 확인
crontab -l
# 백업 스크립트 등록 : 매일 02:00 에 수행
# cat <(crontab -l) <(echo "0 2 * * * /yourfolder/couchbase/backup.sh") | crontab -
crontab -e

0 2 * * * /yourfolder/couchbase/backup.sh

# 등록된 잡 확인
crontab -l

# cron log 확인
tail -f /var/log/cron
# cron job 전체 삭제
crontab -r
```

## Resotre

cbbackup을 사용하여 백업된 데이터를 복원한다.

bucket 을 미리 생성해놔야 한다.

```bash
cd /opt/couchbase/bin/
./cbrestore /yourfolder/couchbase/backups/db-backup-{날짜} http://127.0.0.1:8091 -u id-p pwd
```

### 참고자료

[https://docs.couchbase.com/server/current/backup-restore/backup-restore.html](https://docs.couchbase.com/server/current/backup-restore/backup-restore.html)
[https://docs.couchbase.com/server/current/backup-restore/cbbackupmgr.html](https://docs.couchbase.com/server/current/backup-restore/cbbackupmgr.html)
[https://docs.couchbase.com/server/current/cli/cbtools/cbbackup.html](https://docs.couchbase.com/server/current/cli/cbtools/cbbackup.html)
[https://docs.couchbase.com/server/current/cli/cbtools/cbrestore.html](https://docs.couchbase.com/server/current/cli/cbtools/cbrestore.html)
