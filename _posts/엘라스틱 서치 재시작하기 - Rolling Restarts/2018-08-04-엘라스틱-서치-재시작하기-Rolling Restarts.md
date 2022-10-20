엘라스틱 서치 버전 업그레이드 또는 서버 OS 나 하드웨어 업그레이드 등의 이유로 서비스를 중단해야할 경우가 생긴다. 하지만 무중단 서비스를 지향한다면 엘라스틱서치 클러스터 전체가 아닌 노드를 한 번에 하나씩 재시작해야 한다. 영어로 Rolling Restarts라고 하는데, 문제는 엘라스틱 서치는 데이터를 각 노드에 균등하게 배분하고 레플리카 또한 완전한 셋을 유지하고 싶어 한다는 것이다. 그래서 노드 하나가 셧다운되면, 클러스트는 그 상황을 즉시 인식하고 셧다운된 노드에 있는 프라이머리 샤드와 레플리카 샤드를 다른 노드로 옮기는 Reallocation, Rebalancing 작업이 시작된다.

샤드 재배치, 리밸런싱 작업은 고 비용의 IO 작업이다. 하지만, 엘라스틱 서치 클러스는 노드가 어떤 이유로 셧다운되었는지를 모르기 때문에, 무작정 진행할 수 밖에 없다. 그래서 운영자인 우리가 리밸런싱 작업을 막아줘야 한다. 그렇다면 이제 엘라스틱 서치 무중단 재시작 순서를 알아보자.

#### 1. 인덱싱 중지 및 동기화 된 플러시 - Stop Indexing and Synced Flush
샤드는 5 분 동안 인덱싱 작업이 일어나지 않으면 비활성화 되고 이때, 동기화 된 플러시가 일어나면서 비활성화 된 샤드에 sync_id라는 고유 마커를 부여한다. 이게 무슨 말이냐면, 트랜잭션 로그에만 유지되던 데이터를 Lucene에도 영구적으로 유지되게 한다. 즉, 샤드 데이터를 다시 색인할 필요가 없음을 sync_id로 빠르게 확인하기 때문에 복구 시간이 단축된다. 따라서 인덱싱을 중지 할 수 있다면, 중지하고 동기화 된 플러시를 사용하자.
```text
POST /_flush/synced
```

#### 2. 샤드 배치 비활성화 - Disable shard allocation
앞서 언급한 샤드 재배치, 리밸런싱 작업으로 인한 고 비용의 IO 작업을 막는 방법이다.

```text
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"
    }
}
```

#### 3. 노드 셧다운 - Shut down a single node
#### 4. 업그레이드 작업 - Perform a maintenance/upgrade
#### 5. 노드 재시작 및 클러스트 조인 확인 - Restart the node, and confirm that it joins the cluster
#### 6. 샤드 배치 활성화 - Reenable shard allocation
```text
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
}
```
샤드 리밸런싱에 시간이 걸릴 수도 있다. 클러스트 상태가 초록불이 될때까지 기다리자.
#### 7. 각 노드당 2에서 6 반복 - Repeat 2 through 6 sequence for each node
#### 8. 인덱싱 활성화 - Start Indexing
이 단계에서부터 인덱싱을 다시 시작하면 된다. 하지만, 인덱싱을 다시 시작하기 전에 클러스트의 리밸런싱이 완전히 끝난 다음에 시작하면 재시작 절차가 훨씬 빨리 끝날 것이다.

## Reference

* https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-flush.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-synced-flush.html
* https://stackoverflow.com/questions/43398258/where-to-use-flush-and-flush-synced-commands
* https://www.elastic.co/guide/en/elasticsearch/guide/current/_rolling_restarts.html