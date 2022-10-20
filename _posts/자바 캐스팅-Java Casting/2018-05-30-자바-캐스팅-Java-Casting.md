형변환이라 부르기도 하는 Casting은 기본형과 참조형에서 사용가능하다.  같은 종류의 변수끼리 Casting이 가능하고 기본형과 참조형 간의 Casting은 불가능하다. Casting은 단순히 데이터 형을 바꾸는 목적이 아니라, OOP의 특성인 다형성측면에서 활용한다.

## Casting

기본적인 형변환에 대해 기본형 데이터로 알아보자.

```java
public class Casting {
 
    public static void main(String[] args) {
        int intType = 1;
        double doubleType = 2.0;
         
        //암시적 캐스팅
        doubleType = intType;
 
         // 컴파일 에러
         // intType = doulbeType;

        //명시적 캐스팅
        intType = (int)doubleType;
    }
}
```

컴파일러는 기본형 데이터의 크기를 알고 있다. 작은 범위에서 큰 범위로 형변환을 할 때, 데이터 손실이 발생하지 않는다는 것을 안다. 그래서 캐스트 연산자를 명시적으로 작성할 필요가 없고, 컴파일러가 알아서 형변환을 해주면서 컴파일 에러도 발생시키지 않는다. 반대로, 큰 범위의 double 자료형에서 int 자료형으로 형변환을 시도할 경우, 컴파일러는 데이타 손실을 우려하기 때문에 컴파일 에러를 발생시킨다. 하지만, 개발자가 데이터가 손실된 다는 것을 알고 있음에도 형변환을 하고자 할때, 캐스팅 연산자를 명시적으로 작성하는 것이다.

**즉, 변수에 값을 넣을 때, 해당 자료형에 딱 맞게 정보를 넣어줘야 한다.**

## UpCasting

Casting 은 서로 관련있는 데이터형끼리만 변환이 가능하다.  boolean 자료형과 int 자료형간의 형변환은 일어날 수 없다. 참조형 데이터 역시 마찬가지다. 상손관계, 인터페이스 구현으로 관련 여부가 판단된다.

```java
public class Car{
  // doSometing...  
}

public class AudiCar extends Car{
  // doSometing...  
}
```

요런 단순한 상속 구조의 클래스가 있다.

```java
Car justCar = new Car();
```
Car 데이터 형에 Car 인스턴스를 생성한다. 앞서 설명한데로, Car 의 정보를 모두 가지고 있으므로 문제가 없다.

```java
Car implicitCar = new AudiCar();
```

Car 데이터 형에 AudiCar 인스턴스를 생성한다. AudiCar는 Car 를 상속한 클래스로써 Car 의 모든 정보를 가지고 있다. 따라서 오류가 발생하지 않는다. 이렇게 사용해도 되지만, 데이터 형이 다르므로 아래와 같이 명시적으로 캐스팅 연산자를 작성해주는게 좋다.

```java
Car explicitCar = (Car) new AudiCar();
```

앞선 글에서 int 자료형에 double 자료형의 값을 넣을 때 값 손실이 일어남을 봤다. 그렇다면 Car 자료형에 AudiCar 객체를 생성하면 AudiCar의 고유 정보를 잃어버리는 것일까? 그렇지 않다. 그저 AudiCar 의 특성이 가려지는 것이다. 그래서 이를 되돌리는 Casting이 있다.

## DownCasting

부모 자료형으로 형변환하는 것을 UpCasting이라 하고 그것을 다시 되돌리는 것을 DownCasting이라 한다. DownCasting은 명시적으로 타입을 항상 지정해줘야 한다.

```java
Car implicitCar = new AudiCar();
AudiCar audiCar = (AudiCar) implicitCar;
```
DownCasting은 쉽게 성립하는 문법이 아니다. 상단의 코드처럼 처음 Car 인스턴스를 생성할 때, AudiCar 로 생성했기 때문에 DownCasting이 가능한 것이다. UpCasting 된 변수가 가리키는 객체의 진짜 타입은 어떻게 구분할까?

```java
Car implicitCar = new AudiCar();

if (implicitCar instanceof AudiCar) {
    AudiCar audiCar = (AudiCar) implicitCar;
}
```
**레퍼런스가 어떤 객체의 타입인지 구별하기 위해서 instanceof 키워드를 이용한다.**

## 언제 사용하나?

AudiCar 뿐만아니라 BMWCar, BentzCar 등 많은 자식 객체가 생성되었다. 그런데 부모 클래스 Car의 정보와 기능만 필요할 때 UpCasting을 사용한다.

```java
  List<Car> cars = new ArrayList<>();

 cars.stream()...
```

따로 따로 모든 자식 클래스의 인스턴스를 처리는 것보다 UpCasting을 통해 Car 로 묶어서 처리하는게 편하다. 코드를 보면 심지어 List 를 생성할 때도 UpCasting을 사용하고 있다. 우리는 알게 모르게 UpCasting을 사용하고 있는 것이다. 반대로, UpCasting 된 자료형이지만, 자식 자료형의 고유 기능을 사용하고 싶을 때, DownCasting을 사용한다. 말로 설명하기 힘든데, 실제 개발을 하다보면 이해가 될 것이다.

## Reference

* http://mommoo.tistory.com/40?category=577684
* http://inor.tistory.com/40
* https://blog.naver.com/premiummina/220601833463