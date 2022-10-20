스프링 프레임워크 환경에서 reqeust 오브젝트의 요청 URI 정보를 얻고 싶을 때가 있다. reqeust.get 만 타이핑해도 IDE의 인텔리센스에서 get으로 시작하는 메서드를 나열해준다. 거기서 reqeust.getRequestURI() 를 선택하면 요청 URI 정보를 얻을 수 있다. 하지만, 뷰에서 요청을 하면 원하던 값이 아닌 뷰 경로를 돌려준다. 이럴 때는 request.getAttribute(“javax.servlet.forward.request_uri”)를 사용하면 원하는 값을 얻을 수 있다.

```java
reqeust.getRequestURI();
request.getAttribute("javax.servlet.forward.request_uri");
```

이와 관련한 배경지식도 쌓고 둘의 차이도 알아보자.

## 들어가기

### 스프링 MVC 아키텍처

스프링 MVC의 핵심은 프론트 컨트롤러 역할을 하는 DispatcherServlet이다. 프론트 컨트롤러는 프레젠테이션 계층의 제일 앞에서 서버로 들어오는 모든 요청을 먼저 받아서 처리한다. 클라이언트가 보낸 요청을 받아서 공통적인 작업을 먼저 수행한 후에 적절한 세부 컨트롤러로 작업을 위임해준다. 세부 컨트롤러는 사용자의 요청을 받아서 비즈니스 로직을 수행하도록 서비스 계층 오브젝트에게 작업을 위임한다. 그리고 결과를 받아서 모델을 생성하고 어떤 뷰를 사용할지 결정해서 프론트 컨트롤러에게 돌려준다. 뷰와 모델 정보를 받은 프론트 컨트롤러는 뷰 오브젝트에게 모델을 전달해주고 클라이언트에게 돌려줄 최종 결과물을 생성해달라고 요청한다. HTML을 생성해 달라는 것이고 최종적으로 결과물을 HttpServletResponse에 담아 서블릿 컨테이너에게 돌려준다. 서블릿 컨테이너는 HttpServletResponse에 단김 정보를 HTTP 응답으로 만들어 사용자의 브라우저나 클라이언트에 전송하고 작업을 종료한다.

### Servlet과 JSP

이 글 설명에 필요한 특징만 나열하겠다.

#### Servlet

* 자바 기반으로 만드는 웹 어플리케이션 프로그래밍 기술
* 자바 코드 속에 HTML 코드가 들어가는 구조
* MVC 패턴에서 Controller로 이용

#### JSP

* HTML 코드 속에 자바 코드가 들어가는 구조를 갖는 웹 어플리케이션 프로그래밍 기술
* WAS (Servlet Container)에 의하여 JSP를 서블릿 클래스로 변환하여 사용

## 자세히 보기

앞서 설명한 스프링 MVC 아키텍처의 흐름에서 컨트롤러로부터 뷰와 모델 정보를 받은 프론트 컨트롤러 (DispatcherServlet)는 뷰 오브젝트 (JSP)에게 모델을 전달한다. 서블릿 컨테이너는 이 과정에서 한 가지 일을 수행하는데, **RequestDispatcher의 include() 또는 forward()에 의해서 Servlet (DispatcherServlet)에서 다른 Servlet (JSP)으로 이동할 때, 서블릿 컨테이너는 타켓 서블릿이 최초 호출된 것처럼 path 환경을 바꾼다.** 이 것이 뷰에서 엉뚱한 값을 얻는 이유다.

#### Servlet spec 2.4

서블릿 스펙 2.4에는 어떠한 경우에도 타켓 서블릿이 원래 요청 URI를 알 수 있도록 다음의 request attribute가 포함되도록 했다.

```java
request.getAttribute("javax.servlet.forward.request_uri"); 
request.getAttribute("javax.servlet.forward.context_path");
request.getAttribute("javax.servlet.forward.servlet_path"); 
request.getAttribute("javax.servlet.forward.path_info"); 
request.getAttribute("javax.servlet.forward.query_string");

request.getAttribute("javax.servlet.include.request_uri"); 
request.getAttribute("javax.servlet.include.context_path"); 
request.getAttribute("javax.servlet.include.servlet_path"); 
request.getAttribute("javax.servlet.include.path_info"); 
request.getAttribute("javax.servlet.include.query_string");
```

이 덕분에 우리는 forward/include 된 뷰에서도 원래 요청 URI 정보를 얻을 수 있다.

## 마무리

정리하면, 일반적으로 컨트롤러에서는 request.getRequestURI()를 사용하여 원하는 값을 얻을 수 있고 request attribute를 사용하면 하면 NULL 값을 받는다. 뷰에서는 request.getRequestURI()를 사용하면 뷰 경로를 얻게 된다. RequestDispatcher가 해당 뷰로 forward하기 때문이다. 뷰에서는 아래의 코드로 얻어야 한다.
```java
String uri = (String)request.getAttribute( "javax.servlet.forward.request_uri" );
```

만약, servlet filter에서 RequestDispatcher의 forward()를 호출 했다면 컨트롤러에서는 어떻게 될까? 이런 경우에도 request attribute를 사용해서 요청 URI 정보를 가져와야 한다.

## Reference
* <http://b.pungjoo.com/entry/getURI%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0>
* <http://til0804.tistory.com/25>
* [토비의 스프링 3.1 vol 2](http://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)
