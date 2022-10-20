카우치베이스 커뮤니티버전 6.5.1 설치한 내용을 정리했다.

# linux settings

Swappiness = 0

vm.swappiness = 0 스왑 사용안함

vm.swappiness = 1 스왑 사용 최소화

vm.swappiness = 60 기본값

vm.swappiness = 100 적극적으로 스왑 사용

```bash
# swappiness 설정 확인
cat /proc/sys/vm/swappiness
or
sysctl vm.swappiness

# 즉시 변경
sysctl -w vm.swappiness=0
# 영구 적용
vi /etc/sysctl.conf
vm.swappiness = 0
```

THP 비활성화

> [https://access.redhat.com/solutions/46111](https://access.redhat.com/solutions/46111)

```bash
# 설정 확인
cat /sys/kernel/mm/transparent_hugepage/enabled
cat /sys/kernel/mm/transparent_hugepage/defrag

# 즉시 변경
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

영구 변경

```bash
# root 계정 전환
su - root

vi /etc/init.d/disable-thp

# 스크립트 생성
#!/bin/bash
### BEGIN INIT INFO
# Provides:          disable-thp
# Required-Start:    $local_fs
# Required-Stop:
# X-Start-Before:    couchbase-server
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Disable THP
# Description:       Disables transparent huge pages (THP) on boot, to improve
#                    Couchbase performance.
### END INIT INFO

case $1 in
  start)
    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/transparent_hugepage
    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
    else
      return 0
    fi

    echo 'never' > ${thp_path}/enabled
    echo 'never' > ${thp_path}/defrag

    re='^[0-1]+$'
    if [[ $(cat ${thp_path}/khugepaged/defrag) =~ $re ]]
    then
      # RHEL 7
      echo 0  > ${thp_path}/khugepaged/defrag
    else
      # RHEL 6
      echo 'no' > ${thp_path}/khugepaged/defrag
    fi

    unset re
    unset thp_path
    ;;
esac

# script executable
chmod 755 /etc/init.d/disable-thp

# run the script on boot
chkconfig --add disable-thp

# check tuned-adm active 하고 활성화된 profile 값 기억
tuned-adm active

mkdir /etc/tuned/no-thp
vi /etc/tuned/no-thp/tuned.conf

# 스크립트 작성
[main]
include=활성화된 profile

[vm]
transparent_hugepages=never

# 적용
tuned-adm profile no-thp

# reboot
reboot

# 적용 확인
cat /sys/kernel/mm/transparent_hugepage/enabled
cat /sys/kernel/mm/transparent_hugepage/defrag

# 결과가 아래와 같이 나와야 함
always madvise [never]
```

# Install

```bash
# root 계정 전환
su - root

# install rpm 다운로드
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/bzip2-1.0.6-13.el7.x86_64.rpm

# installed check - bzip2 가 설치되어 있으면 couchabse 만 설치
rpm -qa | grep bzip2
wget https://packages.couchbase.com/releases/6.5.1/couchbase-server-community-6.5.1-centos7.x86_64.rpm
rpm -ivh bzip2-1.0.6-13.el7.x86_64.rpm

# dependencies check - 설치 실패할 때, 필요한 dependencies 파악용도
rpm -qp <couchbase-server-rpm>.rpm --requires

# 카우치베이스 설치
rpm -ivh couchbase-server-community-6.5.1-centos7.x86_64.rpm

# console ui 확인
curl http://127.0.0.1:8091

# start & stop & status
cd /opt/couchbase/bin

# start
./couchbase-server \-- -noinput -detached
# status
./couchbase-server --status
# stop
./couchbase-server -k

# running 상태 확인
./couchbase-server --status
# shutdown
./couchbase-server -k

# couchbase 데이터 폴더 생성
mkdir /yourfolder/couchbase
mkdir /yourfolder/couchbase/data
mkdir /yourfolder/couchbase/backups
cd /yourfolder/
chown -R couchbase:couchbase couchbase
```

Systemd 등록

```bash
# rpm 설치 시, 자동 생성된 스크립트 확인
cat /lib/systemd/system/couchbase-server.service

# 없으면 직접 작성
# -*- mode: conf-unix; -*-
[Unit]
Description=Couchbase Server
Documentation=http://docs.couchbase.com
After=basic.target local-fs.target network.target remote-fs.target nss-lookup.target
Requires=basic.target local-fs.target
Wants=saslauthd.service

[Service]
SyslogIdentifier=couchbase
User=couchbase
Type=simple
WorkingDirectory=/opt/couchbase/var/lib/couchbase
LimitNOFILE=70000
LimitMEMLOCK=infinity
ExecStart=/opt/couchbase/bin/couchbase-server -- -noinput
ExecStop=/opt/couchbase/bin/couchbase-server -k

[Install]
WantedBy=multi-user.target

# 저장 후, 활성화
systemctl daemon-reload && systemctl enable couchbase-server && systemctl restart couchbase-server
# 확인
systemctl status couchbase-server.service
```

## Port 변경

port 충돌이 일어날 경우, port 변경이 필요하다.

카우치베이스 정지

```bash
systemctl stop couchbase-server.service
```

컨피그 편집

```bash
vi /opt/couchbase/etc/couchbase/static_config
```

port 변경

```bash
{indexer_admin_port, 10000}.
{indexer_scan_port, 10001}.
{indexer_http_port, 10002}.
{indexer_stinit_port, 10003}.
{indexer_stcatchup_port, 10004}.
{indexer_stmaint_port, 10005}.
```

이전 컨피그 삭제

```bash
rm -rf /opt/couchbase/var/lib/couchbase/config/config.dat
```

카우치베이스 시작

```bash
systemctl start couchbase-server.service
```

### 참고 자료

[https://docs.couchbase.com/server/current/install/install-production-deployment.html](https://docs.couchbase.com/server/current/install/install-production-deployment.html)
[https://docs.couchbase.com/server/current/install/rhel-suse-install-intro.html](https://docs.couchbase.com/server/current/install/rhel-suse-install-intro.html)
[https://docs.couchbase.com/server/current/install/thp-disable.html](https://docs.couchbase.com/server/current/install/thp-disable.html)
[https://docs.couchbase.com/server/current/install/install-swap-space.html](https://docs.couchbase.com/server/current/install/install-swap-space.html)
[https://docs.couchbase.com/server/current/install/install-ports.html](https://docs.couchbase.com/server/current/install/install-ports.html)
