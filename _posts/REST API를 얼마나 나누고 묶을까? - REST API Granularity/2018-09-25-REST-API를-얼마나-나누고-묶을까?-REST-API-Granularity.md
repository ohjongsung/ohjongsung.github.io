해석하면 API 입도, 즉 API 를 얼마나 세밀하게 나눌 것이냐(Fine grained), 거칠게 묶을 것이냐(Coarse grained) 를 의미한다.

## Fine grained vs Coarse grained

### Case 1
다음과 같은 REST API 가 있다. 각 API는 서로 다른 API Provider를 통해 제공된다고 하자.

**상품 정보 제공 API (Product Service Provider)**
```
GET /products/{productId}
```
***상품 별점정보  API (Point Service Provider)**
```
GET /products/{productId}/point
```
**상품 판매 순위 정보 API (Rank Service Provider)**
```
GET /products/{productId}rank
```

이제 상품 화면을 담당하는 API Consumer는 화면에 상품, 별점, 순위 정보를 표현하기 위해 총 3번의 API 호출을 해야한다.


```text
GET /products/1 HTTP/1.1
{ productId: 1, product_name : "나이키 운동화", price: 50,000 }

GET /products/1/point HTTP/1.1
4.4

GET /products/1/rank HTTP/1.1
10
```

이런 방식의 접근은 API Consumer가 서버에 여러개의 요청을 하게 한다. 이는 API Consumer는 알아야 하는 end point가 늘어나게 된다. 그래서 API Consumer가 Poduct Service Provider에게 한번에 모든 정보를 다 달라고 요청할 수도 있다.

```text
GET /products/1 HTTP/1.1
{ productId: 1, product_name : "나이키 운동화", price: 50,000, point: 4.4, rank: 10 }
```
한번에 정보를 다 담아서 줬더니, 상품 목록을 담당하는 API Consumer가 point 정보는 제외하고 달란다. 여기서 Provider의 선택지는 보통 이렇다. 그냥 불필요한 정보도 가져다 쓰라고 하거나, 특정 필드만 추가로 받을 수 있게 include 패턴같은 것을 적용해주거나. 즉, 생산성, 품질 등을 고려해서 Consumer와 Provider 중 설득력있는 쪽이 이긴다. 그리고 Provider 중에서도 Consumer와 가까운(URI 계층구조 상위) Provider가 독박 쓸 확률이 높다.

### 어떻게 풀어낼까?
앞선 내용을 요약하면, API가 Fine grain 할 수록 Consumer의 API 호출 빈도수가 늘어난다는 것이다.그리고 Consumer가 도메인에 대한 지식을 갖거나 비지니스 로직을 수행해서는 안된다는 것이다. 이 문제는 Consumer 입장에서 해결해줘야 한다. Consumer가 알아야하는 API가 많을 수록 생산성과 서비스 품질이 떨어진다. 일반적으로 하나의 큰 Request보다 작지만 많은 Request가 Latency를 더 높힌다. 즉,  호출해야하는 API의 개수를 줄여주는 것이다.

그렇다면 어떻게 풀어내야할까? 일반적으로 API Composition을 두는 것이다. 중간에 Consumer의 요청을 받아서 필요한 모든 정보를 각 Provider에게 요청해서 받고 Composition해서 돌려주는 것을 말한다.

### API Composition Layer 위치
보통 API GateWay 앞, 내부, 뒤 이렇게 세 곳 중 한 곳에 위치한다. 각 장단점을 살펴보자.

#### API Gateway 앞
* Wrapper 형태로 Gateway 앞에 위치해서 Gateway 단을 전혀 건드리지 않는다.
* Composition Layer가 Consumer의 End Point가 되면서 모든 트래픽을 부담해야 한다.
* Composition Layer와 Gateway 간의 Latency를 최소화해야 한다.
* Composition Layer가 필요하지 않아도 무조건 통과해야 한다.

#### API Gateway 내부
* Gateway 내부에서 동작함으로 Network Hop이 존재하지 않는다.
* Gateway 구현이 복잡해지고, Composition Layer 때문에 성능에 영향을 줄 수도 있다.
* 로직 변경이나 확장이 필요할 때, Gateway 재시작이 필요할 수도 있다.

#### API Gateway 뒤
* Gateway는 아무것도 하지 않아도 된다. 그래서 도입하기 가장 쉽다.
* 사실상 Composition Layer가 API Provider가 되는거나 마찬가지다.
* Composition Layer를 위한 End Point를 API Provider가 제공해야 할 수도 있다.

### Composition 로직 작성
#### Declarative
마크업 형태의 설정을 사용하는 형태로 예를 들면 다음과 같다.

```xml
<Api path="/products/{product_id}/summary">
  <Parallel>
    <Invoke path="/products/{product_id}" />
    <Invoke path="/products/{product_id}/point" />
    <Invoke path="/products/{product_id}/rank" />
  </Parallel>
</Api>
```

#### Imprerative
스크립트 코드로 작성하는 형태로 예를 들면 다음과 같다.

```js
function getProductSummary(product_id) {
  async.parallel([
    function(callback) {
       invoke(function() {
          callback(null, "/products/{product_id}");
       }
    },
    function(callback) {
       invoke(function() {
          callback(null, "/products/{product_id}/point");
       }
    },
    function(callback) {
       invoke(function() {
          callback(null, "/products/{product_id}/rank");
       }
    }
  ]};
}
```

#### API
현 직장에서는 Composition API를 만들어서 제공하고 있다. Gateway 뒤에 Consumer 별 API를 만들어서 필요한 Service 의 정보를 Composition 해서 제공하고 있다. 이렇게 해서 Consumer 별로 소수의 End Point를 유지하면서 원하는 형태의 데이터를 제공받게 된다.

### Case 2
은행 계좌에 현금을 입금하는 경우를 생각해보자. 여기에 필요한 API는 무엇일까? 단순히 비지니스 규칙을 적용해서 간단히 나열해보면 다음과 같다.
* 입금액이 허용 한도 내인지 확인
* 고객 계정 잔액 업데이트
* 고객에게 모바일 또는 문자 등으로 알림 전송
* 트랜잭션 항목 추가

이제 API Provider는 앞서 나열한 3 가지의 비지니스 규칙에 맞게 API를 CRUD에 맞춰서(입금취소, 잔액부족 등) 제공한다고 해보자. 그러면 Consumer는 고객이 입금을 하면 각 API를 호출하면서 트랜잭션도 고려해야하고 무엇을 먼저 호출해야 하는지 등도 생각해야 한다. 그리고 고객에게 알림 전송은 보통 비동기처리를 하므로 MQ에 결과를 전송하는 것도 해야한다. 즉, 비지니스 로직을 알야한다는 것이다.

어떻게든 비지니스 로직을 파악하고 제공되는 API를 사용해서 클라이언트 APP을 만들어 공개했다고 보자. 그런데 비지니스 로직의 변경이 발생했다. 새로운 version의 APP을 만들어서 APP Store에 올렸지만, 고객은 업데이트를 하지 않는다. 어떻게 해야할까?

HTTP Method (GET, POST, PUT, DELETE) 를 임의의 외부 Consumer가 마치 Database의 CRUD처럼 혼동해서 사용하게 해서는 안된다. Consumer가 도메인 지식을 파악하고 조작하게 해서는 안되는 것이다.

### 어떻게 풀어낼까?
CRUD를 숨기고 Consumer에게 입금이라는 하나의 API 정도로 거칠게 묶어서(Coarse grain) 제공하는 것이다. 이렇게 하면 고객 계좌의 상태 변경은 서비스 제공자만 할 수 있게 된다.

## 마무리
나는 한때 Composition API를 만든 팀에서 일했었다. REST API에 대해서 알면 알수록 뭔가 이론에 맞지 않는 API를 만든다는 생각에 의문이 들었다. 그런던 중에 Composition Layer의 존재를 알게 되었고 내가 하던 일이 무엇인지 정확히 알게 되었다. 시간이 지나면 잊어먹을까 염려되어 이렇게 정리해 본다.

## Reference

* https://blog.naver.com/saltynut/120210979446
* https://www.thoughtworks.com/insights/blog/rest-api-design-resource-modeling