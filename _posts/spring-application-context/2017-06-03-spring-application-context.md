스프링 애플리케이션에서는 독립된 컨테이너가 오브젝트에 대한 제어권을 가지고 있다. 그래서 IOC 컨테이너라고 부른다. 그런데 이 컨테이너를 빈 팩토리 또는 애플리케이션 컨텍스트라고도 부른다. 왜 그렇게 부르는지 소스를 살펴보자.

#### ApplicationContext interface

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory
, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
```

우리가 말하는 IOC 컨테이너는 ApplicationContext 인터페이스를 구현한 클래스의 오브젝트이다. 그런데
소스를 보면 ApplicationContext 인터페이스는 BeanFactory 인터페이스의 서브 인터페이스인 ListableBeanFactory, HierarchicalBeanFactory를 상속하고 있다. 즉 애플리케이션 컨텍스트가 IOC 컨테이너이자 빈 팩토리이며, 그 이상의 기능을 가졌다고 볼 수 있다.

## 계층구조 컨텍스트

스프링 프로젝트에서 일반적으로 ApplicationContext를 부모 컨텍스트인 RootContext와 자식 컨텍스트인 ServletContext 나눠서 계층 구조로 만든다. 계층구조 안의 모든 컨텍스트는 각자 독립적인 설정정보를 이용해 빈 오브젝트를 만들고 관리한다. 계층구조 컨텍스트의 특징은 다음과 같다.

#### 계층구조 컨텍스트의 특징

* 자식 컨텍스트는 자신이 관리하는 빈에서 필요한 빈이 없는 경우, 부모 컨텍스트의 빈까지 검색한다. 부모 컨텍스트에서 원하는 빈이 없다면, 부모의 부모 컨텍스트를 거쳐 최상위 루트 컨텍스트까지 원하는 빈을 검색한다.
* 어디까지나 자신의 부모 컨텍스트에게만 빈 검색을 요청할 수 있다. 자식 컨텍스트에게는 요청할 수 없다. 또한 형제 컨텍스트의 빈도 검색할 수 없다.
* 부모 컨텍스트와 같은 이름의 빈을 정의해서 가지고 있다면 자신이 가진 빈을 우선해서 사용한다. 즉, 부모 컨텍스트가 정의한 것은 무시된다.
* 하나의 컨텍스트에서 설정된 AOP는 다른 컨텍스트에 등록된 빈에 영향을 미치지 않는다.

#### 애플리케이션 컨텍스트에 빈 오브젝트를 등록

일반적으로 서블릿 컨텍스트에선 @Controller 빈을 스캔해서 등록하고, 루트 컨텍스트에선 그 외 나머지 빈을 등록하도록 스캔을 설정한다.

#### RootContext Configuration
```java
@ComponentScan(basePackages = {"io.ohjongsung.template"}
,excludeFilters = @ComponentScan.Filter(value = Controller.class, type = FilterType.ANNOTATION))
```
```xml
<context:component-scan base-package="io.ohjongsung.template">
<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
</context:component-scan>
```

#### ServletContext Configuration
```java
@ComponentScan({ "io.ohjongsung.template" }
,includeFilters = @ComponentScan.Filter(value = Controller.class, type = FilterType.ANNOTATION))
```
```xml
<context:component-scan base-package="io.ohjongsung.template" use-default-filters="false">
<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
</context:component-scan>
```

## 왜 계층구조를 이용할까?

사실 스프링은 하나의 컨텍스트에 모든 설정을 다 할수 있고, 컨텍스트를 나눠놓고 componetScan을 한쪽에서만 한다던가 부모 자식 컨텍스트가 모든 빈을 중복 스캔하게 설정할 수 있다. 계층구조 컨텍스트의 특징을 잘 이해하고 설정한다면 이런 방식도 큰 문제가 없다. 그런데 왜 계층구조를 이용할까?

계층구조는 애플리케이션 구조에 대한 파악과 설정을 편하게 한다. 기존 설정을 수정하지 않고 사용하지만 일부 빈 구성을 바꾸고 싶은 경우, 애플리케이션 컨텍스트를 계층구조로 만들면 편하다. 자식 컨텍스트를 생성하고 바꾸고 싶은 빈을 다시 설정하면 된다. 이해가 안된다면, 계층구조 컨텍스트의 특징을 다시 읽어보자. 또 다른 이유는 여러 컨텍스트가 공유하는 설정을 만들기 위해서이다. 애플리케이션 안에 성격이 다른 설정으로 분리해서 여러 컨텍스트를 구성하고 이 컨텍스트들의 부모 컨텍스트를 만들어서 공통 설정은 공유하게 만들 수 있다.

## AOP 설정 주의사항

스프링 AOP는 데코레이터 패턴 또는 프록시 패턴을 응용해서, 기존 코드에 영향을 주지 않고 부가기능을 타깃 오브젝트에 제공할 수 있는 객체지향 프로그래밍 모델이다. 이는 여러 오브젝트에 필요한 공통적인 기능을 손쉽게 개발하고 적용할 수 있다. 그렇다면, 부모 컨텍스트에 등록된 빈에 AOP를 적용했는데 이 빈을 자식 컨텍스트에서 다시 스캔해서 등록하면 어떻게 될까? 계층구조 컨텍스트의 특징 두 개를 다시 보자.

* 하나의 컨텍스트에서 설정된 AOP는 다른 컨텍스트에 등록된 빈에 영향을 미치지 않는다.
* 부모 컨텍스트와 같은 이름의 빈을 정의해서 가지고 있다면 자신이 가진 빈을 우선해서 사용한다. 즉, 부모 컨텍스트가 정의한 것은 무시된다.

위의 두 특징에 따르면, 부모 컨텍스트에 AOP를 적용해도 자식 컨텍스트가 같은 이름의 빈을 정의해서 가지고 있다면 AOP 설정은 무시된다. 그래서 자식 컨텍스트에서 빈을 이유없이 중복 등록하지 않는지 주의해야 하며, AOP 설정이 필요한 빈이 정의된 컨텍스트면 모두 설정 해야한다.