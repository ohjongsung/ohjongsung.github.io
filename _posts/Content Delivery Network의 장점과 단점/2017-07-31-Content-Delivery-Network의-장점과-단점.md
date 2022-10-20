Content Delivery Network (CDN)은 웹 호스팅에서 사용하는 기술이다. 대부분의 CDN은 이미지, 동영상, 미디어, CSS, JavaScript 같은 정적 파일을 호스팅 한다. 그래서 여러 공공 및 사설 CDN에서는 유명한 JavaScript 라이브러리, CSS, 글꼴 등을 서비스하고 있다. 실제로 이 블로그도 CSS와 JavaScript 라이브러리를 CDN에서 가져와서 사용하고 있다.

## CDN Services

구글, 마이크로소프트와 같은 대형 IT 기업들은 많은 무료 CDN 서비스를 하고 있다. 예를 들어 JQuery 라이브러리를 CDN을 통해서 로딩하고 싶다면 다음과 같은 코드로 사용할 수 있다.

```html
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.6.4/jquery.min.js.js"></script>
<script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.6.4.min.js"></script>
```
물론 AWS, Azure 같은 상업용 CDN으로 자신의 파일을 호스팅할 수도 있다.

## CDN 장점

#### Different domains

웹 브라우저는 도메인당 동시에 다운로드할 수 있는 파일의 개수를 제한하고 있다. 일반적으로 동시에 네 개의 파일을 다운로드할 수 있게 하며, 5번째 파일의 다운로드는 먼저 받고 있는 파일 중 하나가 완료되기 전엔 시작되지 않는다. 그런데 CDN 파일은 다른 도메인에서 호스팅 되고 있다. 즉, CDN 서비스를 사용하면 브라우저가 동시에 4개의 추가 파일 다운로드를 할 수 있게 된다.

#### Files may be pre-cached

JQuery 같은 경우, 거의 모든 웹 사이트에서 사용된다고 볼 수 있다. 자신의 사이트를 방문하는 사용자는 이미 CDN 서비스를 사용하는 다른 사이트에 방문했을 가능성이 크며, 그렇다면 파일은 이미 브라우저에 캐시 되어 다시 다운로드할 필요가 없다.

#### High-capacity infrastructures

큰 돈을 들여서 좋은 호스팅 인프라를 구축한다고 해도 구글이나 마이크로소프트만큼의 용량이나 확장성이 있다고 할 수 없다. 그래서 CDN을 사용하면 높은 가용성, 낮은 네트워크 대기시간 및 낮은 패킷 손실을 보장할 수 있다.

#### Distributed data centers

웹 서버가 서울을 기반으로 하는 경우, 유럽이나 북미 사용자는 파일에 접근하기 위해 꽤나 긴 네트워크 통신을 해야 한다. 그러나 CDN은 각 대륙이나 대도시에 데이터 센터를 만들어 사용자의 지역에 맞춰 빠른 다운로드 속도를 지원한다.

#### Built-in version control

일반적으로는 특정 버전의 CSS, JavaScript 라이브러리를 CDN에서 가져올 수 있으며, 원한다면 최신 버전을 가져오게 할 수도 있다.

#### Usage analytics

많은 상용 CDN은 일반적으로 바이트 당 요금을 부과하기 때문에 파일 사용 보고서를 제공한다. 이런 보고서는자체 웹 사이트 분석에 도움이 될 수 있으며, 동영상 조회 수 및 다운로드 수를 더 잘 나타낼 수 있다.

#### Boosts performance and saves money

트래픽이 급증하여 서버가 다운되어 손실을 보고, 이를 막기 위해 인프라 구축에 막대한 돈을 투자하는 것보다 CDN을 사용하는 것이 부하를 분산하고 대역폭을 절약하며 성능을 향상시키고 기존 호스팅 비용을 절감할 수 있다.

#### Helps improve SEO

구글은 2010년에 웹 사이트 속도가 검색 엔진 순위에 영향을 줄 것이라고 발표했다. 결국 CDN을 사용하면 웹 사이트 속도가 빨라질 것이고 SEO (검색 엔진 최적화)에 도움이 될 것이다.

#### DDoS protection

많은 상용 CDN은 대시 보드에서 설정하거나 또는 기본적으로 DDoS 공격을 완화할 수 있다.

#### Better user experience

속도가 빠르다는 것은 사용자 경험에 정말 중요하다. 이는 방문자의 이탈률을 낮추고 클릭률을 높인다.

## CDN 단점

#### Additional complexity

오프라인으로 작동하지 않으므로 인터넷 연결 없이는 개발할 수 없다. 그리고 라이브 서버에 배포할 때, 수작업이 필요할 수 있다.

#### Blocked access

간단히 말해서 구글에서 제공하는 무료 CDN 서비스가 중국에서 될까? 글로벌 서비스를 지향한다면 이런 부분을 고려해야 한다.

#### Security

무료 CDN을 사용하면 파일 호출에 자신의 사이트 정보도 함께 전송된다. 이게 싫으면 무료 CDN을 사용하면 안된다. 또한, JavaScript 라이브러리에 자신의 사이트 정보를 수집하는 코드가 삽입되어 있을 수도 있다.

#### Disaster for testing

CDN이 호스팅 된 파일을 자동으로 변경하거나 신뢰할 수 없는 것으로 판단하면, 이를 디버깅하기 어려울 수 있다.

#### Built-in version control

일반적으로는 특정 버전의 CSS, JavaScript 라이브러리를 CDN에서 가져올 수 있으며, 원한다면 최신 버전을 가져오게 할 수도 있다. 무턱대고 최신 버전을 가져오게 설정했다가 이전 버전과 호환되지 않아 오류가 발생할 수 있다.

## Reference
* <https://www.sitepoint.com/7-reasons-to-use-a-cdn/>
* <https://www.keycdn.com/support/7-reasons-you-should-use-a-content-distribution-network/>
* <http://www.techpik.com/7-reasons-why-you-should-use-a-content-delivery-network-cdn/>
* <http://htmlcheats.com/cdn-2/6-reasons-use-cdn/>
* <https://www.sitepoint.com/7-reasons-not-to-use-a-cdn/>





