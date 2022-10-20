Postgresql 커넥션 관련해서 테스트 및 확인할 일이 있어서 로컬에 설치했던 내용을 정리한다.

```bash
docker pull postgres
docker volume create pgdata

docker run -d -p 5432:5432 --name postgres \
   --restart unless-stopped \
   -e POSTGRES_PASSWORD=12345 \
   -v pgdata:/var/lib/postgresql/data \
    postgres:14
```

1. –name postgres-container : 컨테이너 이름 지정.
2. –restart unless-stopped : 따로 stop 명령을 보내지 않으면 컨테이너 재시작의 경우에도 자동 실행됨.
3. -p 5432:5432 : postgreSQL이 사용할 포트 번호 기재
4. -e POSTGRES_PASSWORD=123456 : postgreSQL DB에 대한 비밀번호 설정.
5. -v 볼륨 정보 : 컨테이너가 사용할 볼륨 정보 기재.
6. postgres:14 : 실행할 postgreSQL 컨테이너 버전.

컨테이너를 생성할 때 기본 사용자명이나 기본 데이터베이스 이름를 변경할 수 있다.
아래 환경 변수들을 사용해서 기본 값을 지정할 수 있다.

|---|---|
| POSTGRES_USER | 사용자명 설정 (기본값: postgres) |
| POSTGRES_DB | 기본 데이터베이스 이름 설정 (기본값: POSTGRES_USER 값) |


둘 다 입력하지 않으면 postgres라는 이름으로 기본 사용자와 데이터베이스가 각각 생성됩니다.

### postgresql 진입

```bash
# 한번에 진입
docker exec -it postgres psql -U postgres

# 두단계 진입
docker exec -it postgres /bin/bash
root@caa78328d590:/# psql -U postgres

# 나가기
/q
```

### 새로운 계정 생성

```bash
CREATE USER test PASSWORD '1234' SUPERUSER;
alter user test with createdb;

postgres=# \du
```

### 새로운 DB 생성

```bash
create database testdb; 
grant all privileges on database testdb to test;

\l

\connect testdb
```

### 컨테이너 셧다운 및 삭제

```bash
docker stop postgres
docker rm postgres
```

이렇게 작업하고 다시 컨테이너를 뛰웠을 때, 계정정보가 지속되는지 확인해보자.

### 커넥션 URL 양식

```
"postgresql://postgres:12345@localhost:5432/postgres"
“postgresql://{아이디}:{비번}@{IP주소}:{포트}/{database이름}”
```

## Reference

* https://devinlife.com/postgresql/run-postgresql-on-docker/
* https://judo0179.tistory.com/96
* https://velog.io/@haeny01/Docker-Docker-X-PostgreSQL-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EC%97%B0%EB%8F%99
* https://kanoos-stu.tistory.com/23
* https://psychoria.tistory.com/783