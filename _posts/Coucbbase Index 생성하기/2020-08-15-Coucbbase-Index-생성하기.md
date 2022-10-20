## Primary index 생성

버킷을 만들고 쿼리(n1ql) 을 사용하려면, Primary index를 생성해줘야 한다.

[https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/createprimaryindex.html](https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/createprimaryindex.html)

### format

```bash
CREATE PRIMARY INDEX [index_name]
    ON named_keyspace_ref
    [ USING GSI ]
    [ WITH {"nodes": ["node_name"], "defer_build":true|false}, "num_replica": num_replica_num } ];
```

### example

> {"defer_build":true} : 인덱스 생성을 지연 시켜서, 인덱스 생성 부하를 막는다.
따라서, 생성하려는 인덱스 쿼리를 모두 만들고 미리 실행시킨다.

```bash
CREATE PRIMARY INDEX `temp-index` ON `temp` USING GSI WITH {"defer_build":true};
CREATE PRIMARY INDEX `temp-log-index` ON `temp-log` USING GSI WITH {"defer_build":true};
```

### 생성 확인

```bash
SELECT * FROM system:indexes WHERE name="abuse-blacklist-index";
```

### 생성 시작

클러스터가 여유로운 시간에 인덱스 생성 시작

```bash
BUILD INDEX ON `temp`(`temp-index`) USING GSI;
BUILD INDEX ON `temp-log`(`temp-log-index`) USING GSI;
```

## Secondary index 생성

[https://docs.couchbase.com/server/current/learn/services-and-indexes/indexes/global-secondary-indexes.html](https://docs.couchbase.com/server/current/learn/services-and-indexes/indexes/global-secondary-indexes.html)

Document 에 포함된 field 를 추가해서 index 생성

### date, type field 로 index 생성

```bash
CREATE INDEX `temp-log-date-index` ON `temp-log`(date) USING GSI WITH {"defer_build":true};
CREATE INDEX `temp-log-date-type-index` ON `temp-log`(date, type) USING GSI WITH {"defer_build":true};

BUILD INDEX ON `temp-log`(`temp-log-date-index`) USING GSI;
BUILD INDEX ON `temp-log`(`temp-log-date-type-index`) USING GSI;
```