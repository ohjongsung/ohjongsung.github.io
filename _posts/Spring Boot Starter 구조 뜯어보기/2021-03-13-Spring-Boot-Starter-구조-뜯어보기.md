개발을 하다 보면 수 많은 모듈을 만들고 제공하고 관리하게 된다. 제공하는 쪽이나 받는 쪽이나 버전 관리, 설정 등 이래저래 관리하기 힘들다. 그러다 문뜩 Spring Boot Starter 가 생각이 났고 구조를 뜯어보고 응용할 포인트를 찾아봤다.

Spring Boot 서비스의 의존성을 추가할 때, Spring Boot autoconfigure 와 Spring Boot starter 조합을 사용한다. 버전은 신경쓰지 않고 그냥 groupId 와 artifactId 만 추가하고, application.yml 설정을 하면 사용할 수 있게 된다. 어떻게 이것이 가능한지 하나씩 알아보자.

# autoconfigure

autoconfigure 모듈을 살펴보면, 우리가 사용하지도 않을 수많은 라이브러리가 패키지로 나눠져 있다. 그렇다면 autoconfigure 모듈에는 우리가 사용할지도 모르는 모든 라이브러리들이 미리 준비되어 있는 것일까? 이들을 하나씩 확인해보면, 단순히 configuration 클래스만 있는 것을 알 수 있다.

예를 들어, couchbase 패키지내에 있는 클래스를 보면 CouchbaseAutoConfiguration, CouchbaseProperties 클래스가 있다. 그리고 Conditional 어노테이션으로 실제 동작을 하려면 Couchbase Client 의 특정 클래스가 존재하거나, application.yml 에 spring.couchbase 로 시작하는 설정이 존재해야만 제대로 동작하게끔 구성되어 있다.

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({Cluster.class})
@ConditionalOnProperty({"spring.couchbase.connection-string"})
@EnableConfigurationProperties({CouchbaseProperties.class})
public class CouchbaseAutoConfiguration {
    public CouchbaseAutoConfiguration() {
    }
}
```

```java
@ConfigurationProperties(
    prefix = "spring.couchbase"
)
public class CouchbaseProperties {
}
```

실제 동작 여부는 제어한다고 쳐도 사용할지도 모르는 라이브러리를 미리 다 땡겨놓고 있는 것은 불필요하다. 하지만, 막상 받아놓은 디펜던시를 확인해보면 couchbase 관련 라이브러리가 존재하지 않는다. 이것이 가능한 이유는 선택전 종속성 설정을 했기 때문이다.

## Maven Optional Dependencies

선택적 종속성은 (어떤 이유로 든) 프로젝트를 하위 모듈로 분할 할 수 없는 경우에 사용된다. 일부 종속성이 프로젝트의 특정 기능에만 사용되며 해당 기능을 사용하지 않으면 필요하지 않다는 것이다. 이상적으로 이러한 기능은 코어 프로젝트에 따라 하위 모듈로 분할된다. 이 새로운 서브 프로젝트는 서브 프로젝트의 기능을 사용하기로 결정한 경우 관련 종속이 모두 필요하기 때문에 선택 사항이 아닌 종속성 만 갖는다. 그러나 어떤 이유로 든 프로젝트를 분할 할 수 없을 경우 이러한 종속성은 선택 사항으로 선언된다. 사용자가 선택적 종속성과 관련된 기능을 사용하려면 자신의 프로젝트에서 해당 선택적 종속성을 다시 선언해야한다. 이것이 상황을 처리하는 가장 명확한 방법은 아니지만 선택적 종속성과 종속성 제외는 임시방편이다.

### optional dependencies 사용 이유

1. 선택적 종속성은 공간과 메모리를 절약
2. 라이센스 문제나 Class path 문제를 예방

### optional tag 사용방법

종속성에 <optional> 요소를 추가하고 true 로 설정한다.

```xml
<project>
  ...
  <dependencies>
    <!-- declare the dependency to be set as optional -->
    <dependency>
      <groupId>sample.ProjectA</groupId>
      <artifactId>Project-A</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <optional>true</optional> <!-- value will be true or false only -->
    </dependency>
  </dependencies>
</project>
```

### optional dependencies 동작 및 예시

Project-A (autoconfigure) → Project-B (couchbase)

Project-A 가 Project-B 에 의존할 때, A 가 POM 에 B 를 선택적 종속성으로 선언해도 A 의 종속성 관계에 문제는 발생하지 않는다. Project-A 의 class path 에 Proejct-B 가 추가되는 일반적인 Build 와 똑같이 동작한다.

Project-X (real service) → Project-A (autoconfigure)

Project-X 가 POM 에 Project-A 를 종속성 추가하면, 선택적 종속성 선언 효과가 발생한다. Project-B 는 Project-X 의 class path 에 포함되지 않는다. Project-X 의 POM 에 직접 Proejct-B 의 종속성을 추가해야 한다.

## Starter

이제 Starter 가 무슨 역할을 할지 자연스레 알게 된다. 바로 Project-X 에 직접 종속성을 추가하는 역할을 하는 것이다.

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-couchbase</artifactId>
</dependency>
```

spring-boot-starter-data-couchbase 를 추가하지 않으면, autoconfigure 안의 couchbase 설정 클래스들은 동작하지 않는다. 왜냐하면, 종속성 클래스들이 class path 에 존재하지 않기 때문이다.

## 응용하기

위의 개념을 이해하면, 사내 프레임워크 모듈이나 팀 단위 모듈을 제공하거나 사용하기 편해진다.

xxx-spring-boot-autoconfigure 모듈을 만들고, 제공하고 있는 모든 모듈의 configuration 클래스를 작성한다. 그리고 위에서 생략했지만, application.yml 설정이 필요도 없게끔 각종 설정값을 미리 넣어놓던가 아니면 Configuration Metadata 를 추가해서 application.yml 작성의 편의성을 더욱 증가시킬 수 있다. 그리고 모듈을 가져다 쓰는 곳은 starter 만 추가하면 쉽게 사용할 수 있게 된다.

물론, 사내에는 Spring Boot 가 아닌, 레거시도 있고 이미 사용중인 모듈도 있어서 쉽게 구조를 바꾸기는 어렵다. 일단 조금씩 바꿔봐야겠다.