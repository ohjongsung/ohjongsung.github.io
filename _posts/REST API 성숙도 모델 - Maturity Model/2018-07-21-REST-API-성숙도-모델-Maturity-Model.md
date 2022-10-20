[지난 글](https://ohjongsung.github.io/REST-API-기본-정리/)에 이어서, 이번에는 [Rechardson 의 API 성숙도 모델](https://martinfowler.com/articles/richardsonMaturityModel.html)에 대해서 정리해 보겠다. 어떤 글에서는 6 단계로 나누기도 하는데, 원문의 모델은 4 단계로 나누어져서 각 단계를 달성할 수록 REST API에 가까워진다고 말한다.

## Level 0 : 1 URI, 1 HTTP method
0 단계에서는 API 서비스를 위해 단 하나의 endpoint 를 사용한다. 그리고 전달되는 서로 다른 매개변수를 통해서 하나의 endpoint로 여러 동작을 하게 된다. 매개변수를 Body 로 전달하기 위해 HTTP method 는 POST 가 된다. SOAP, RPC, GraphQL APIS 은 API 성숙도 모델에서 0 단계라고 할 수 있다.
**Request**
```text
POST https://api/userService
{
  "function": "getUserInfo",
  "arguments" [
    "1"
  ]
}
```
**Response**
```text
HTTP/1.1 200 OK
{
  "result" {
    "name": "Ohjongsung",
    "id": "1"
  }
}
```
**CRUD**
```text
CREATE : POST /api/userService
READ :   POST /api/userService
UPDATE : POST /api/userService
DELETE : POST /api/userService
```
## Level 1 : N URI, 2 HTTP method
1 단계에서는 REST API 구성 요소중 Resource 를 도입한다. Resource 별로 고유한 URI를 사용해서 식별하는 것이다. 그러나 아직까지는 HTTP protocol  전부를 활용한다고 볼 수 없다.
**Request**
```text
POST https://api/users/create
{
  "name": "Ohjongsung"
}
```
**Response**
```text
HTTP/1.1 200 OK
{
  "result" {
    "error": "already exist member"
  }
}
```
**CRUD**
```text
CREATE : POST /api/users/create
READ :   GET /api/users/1
UPDATE : POST /api/users/update
DELETE : POST /api/users/remove/1
```
예제를 살펴 보면, HTTP method는 GET 과 POST 만 사용하고 있다. 그리고 Status Code 를 error 인 상황에도 200 OK 로 전달하고 있다. 마지막으로 HTTP headers 에 Content-Type 이나 Cache 관련 정보를 전혀 제공하고 있지 않다.

## Level 2 : N URI, 4 HTTP method
REST에 대해서 조금이라도 안다면, Resource 를 URI 로 식별하고, HTTP method 를 사용해서 CRUD 를 표현한다는 것을 알 것이다. 2 단계에서는 HTTP method 인 GET, POST, PUT, DELETE 를 사용해서 CRUD 를 나타내며 메시지에 Status Code 도 담겨 반환된다. 성숙도 2 단계에서 GET 은 상태를 변화시키지 않는 안전한 Action 이 된다. GET 은 안전하게 어떤 순서로든 얼마든지 호출할 수 있고, 매번 같은 결과(멱등성)를 얻도록 한다. 즉, Cache 를 적용해서 사용자 입장에서 성능 향상을 느낄 수 있다.
**Request**
```text
PUT https://api/users
{
  "name" [
    "Ohjongsung"
  ]
}
```
**Response**
```text
HTTP/1.1 201 Created
Content-Type: application/json
{
  "result" {
    "name": "Ohjongsung",
    "id": "1"
  }
}
```
**CRUD**
```text
CREATE : POST /api/users
READ :   GET /api/users/1
UPDATE : PUT /api/users/1
DELETE : DELETE /api/users/1
```
성숙도 2 단계에서는 더 이상 URI 에 action 이 담기지 않는다. 그리고 멱등성을 보장하는 GET 에는 Cache 가 적용되며, Response 에 HTTP Status Code 가 의미있게 반환된다. HTTP Status Code 를 모두 활용할 필요는 없다. 클라이언트 개발자의 입장에서 자신의 요청이 성공(200 OK) 인지 실패인지만 알고 싶어하는 경우가 많다. 실패라면 클라이언트 잘못(400 Bad Request) 인지 서버 잘못(500 Internal Server Error) 정도면 충분하다고 한다. 다만, 실패라면 Body 에 왜 실패했는지에 대한 정보를 보내 주는 것이 좋다.

## Level 3 : Hypermedia As Engine of Application State
[3 단계까지 달성해야 진정한 REST API](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven) 라고 Roy Thomas Fielding 이 말했다. 하지만 나는 3 단계를 적용한 API를 본적이 아직까진 없다. 성숙도 3 단계는 API 서비스의 모든 endpoint 를 최초 진입점이 되는 URI 를 통해 Hypertext Link 형태로 제공하는 것이다. 하지만, 대부분의 요즘 API 는 잘 작성된 API 문서를 제공하고 있다. 그러나 성숙도 3 단계는 단순히 API 목록 제공뿐만 아니라, 어떤 request 의 다음 request 에 필요한 endpoint 를 제공한다는 것이다. 이렇게 함으로써 서버는 클라이언트에 문제를 일으키지 않고 URI scheme 를 변경할 수 있게 된다. 그리고 클라이언트에게 다음에 어떤 동작이 가능한지 힌트를 제공한다. 어디까지나 클라이언트 개발자가 Body 에 담겨오는 Hypertext Link 를 눈여겨본다는 전제하에 가능하다.
**Request**
```text
GET https://api/
```
**Response**
```text
HTTP/1.1 200 OK
Content-Type: application/json
{
  "/api/users",
  "/api/users/{userId}/roles",
  "/api/products",
  "/api/..."
}
```
**Request**
```text
GET https://api/users/1
```
**Response**
```text
HTTP/1.1 200 OK
Content-Type: application/json
{
 "result" {
    "name": "Ohjongsung",
    "id": "1",
    "nextActions": {
       "/api/users/{userId}/roles",
     }
  }
}
```
성숙도 3 단계에서는 API 를 통해서 해당 API 의 모든 리소스 식별 정보를 파악할 수 있다. 어떻게 보면 REST API 의 제약 조건 중 하나인 Uniform interface 의 self-descriptive messages 를 달성한다고 볼 수 있다.

## Level 4 : API Versionning
3 단계가 Rechardson API 성숙도 모델의 끝이지만, [이 글](https://damienfremont.com/2017/11/23/rest-api-maturity-levels-from-0-to-5/) 을 읽고 추가하기로 했다. 버전 정보는 성숙도 모델 어떤 단계에서도 적용이 가능해서 4 단계라고 할 순 없지만, self-descriptive messages 측면에서 필요하다고 본다.
**Request**
```text
GET https://api/v1/users/1
```
**Request**
```text
GET https://api/users/1
Accept: application/vnd.app-1.0+json
```
**Response**
```text
HTTP/1.1 200 OK
Content-Type: application/json
{
  "result" {
    "name": "Ohjongsung",
    "id": "1"
  }
}
```
예제를 보면, 두 가지 방식으로 request 를 보내고 있다. 첫 번째는 URI 에 버전 정보를 정의하는 것이고 두 번째는 Accept 헤더에 버전 정보를 정의하는 방식이다. 일반적으로 첫 번째 방식을 많이 사용한다.

## Reference 
* https://damienfremont.com/2017/11/23/rest-api-maturity-levels-from-0-to-5/
* http://jinson.tistory.com/190
* https://blog.goodapi.co/api-maturity-fb25560151a3
* https://blog.naver.com/saltynut/220758336130