데이터 캡슐화는 데이터를 공용 메소드를 통해서만 접근하도록 허용하는 방법을 말한다. 그렇다면 getter/setter를 사용하면 얻게 되는 기능은 무엇일까? 정리해보면 다음과 같다.

#### getter/setter의 기능

* 변수에 새로운 값을 할당할 때, Validation 검사 (추후에 추가 되더라도 부담이 없다.)
* lazy loading
* read/write 권한 설정
* DI 또는 테스트를 위한 가변 필드 설정
* 런타임 시, 속성 변경에 대한 디버깅 포인트
* 확장 클래스에서의 재정의
* 대체 속성을 노출 (예를 들면, getAddress()만 노출하고 이를 조합하는 여러 필드 정보는 숨긴다.)


그런데, 이런 기능을 위해 생성한 getter/setter가 전체의 몇% 일까? 내 경험에는 의미 없이 생성한 게 절대 다수이다.

#### 데이터 캡슐화라는 이유로 무조건 getter/setter를 작성하는 것이 옳은건가?

나는 개발 입문을 C#으로 했다. C#에는 Property, Auto Property 같은 개념이 있는데, 접근자 메서드가 굳이 필요하지 않다면 일반 변수처럼 사용이 가능하다. 즉, getter/setter가 필요 없는 경우에는 굳이 작성할 필요가 없다는 것이다. 그런데 JAVA에서는 getter/setter를 생성하는 것을 권장한다. 생성하지 않으면, IDE에서 warning 메시지가 꼴보기 싫을 정도 보인다. 그래서 만들어 놓으면, 의미없는 코드에 또 짜증이 난다. 물론 [Lombock](https://projectlombok.org/features/Data)의 어노테이션을 사용해서 해결할 수 있다. 그래도 JAVA가 C#보다 getter/setter 구현 방식이 구리다는 생각이 든다.

#### VO (Value Object)

```java
class Person {
    private int age;
    private String name;
    public Person() {}
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

```
위의 코드에는 아까 설명한 getter/setter의 기능이 사용되고 있지 않다. 바로 요놈이 무의미한 getter/setter 남발의 원인이 아닐까 한다. 오로지 데이터 저장을 위한 자료구조이다. 여기서 사용한 getter/setter는 캡슐화를 위한다기 보다는 컨벤션이지 싶다.

#### Public 변수 사용으로 바꿀까?

우리나라는 ORM을 쓰지 않고 MyBatis같은 SQL Mapper를 이용한 개발이 많다. 그래서 VO의 주된 개념인 불변 객체로써 활용하지 않고 DB 쿼리를 위한 매개변수가 되기도 하고, DB 쿼리의 결과값도 담고, 비지니스 계층, 프레젠테이션 계층을 돌아다니며 레이어, 티어, 모듈 등의 결합도를 높인다. 이것도 문제긴 한데 getter/setter에 대해 작동하도록 설계된 라이브러리나 프레임워크가 많다. MyBatis도 getter/setter를 통해 ResultMap을 속성과 매핑한다. 그러니까 getter/setter 안만들기가 힘들다. 그래서 getter/setter를 데이터 캡슐화를 위해서만 사용한다고 할 수 없다. 왜 사용하는지 알고 쓰자.

## Reference
* <http://egloos.zum.com/aeternum/v/1232020>
* <https://stackoverflow.com/questions/1568091/why-use-getters-and-setters>
* <http://lacti.me/2011/10/03/getter-and-setter-at-java>
* <http://qna.iamprogrammer.io/t/encapsulation-getter-setter/193/27>


