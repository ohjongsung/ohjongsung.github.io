모던 자바는 자바5 이후를 말한다. 그 대표적인 특징은 세 가지로 애노테이션, 람다 그리고 제네릭이다. 이번 글로 알게 모르게 사용하고 있는 제네릭에 대해서 정리해보자.

## 사전 정의

generic
> 1. formal shared by, typical of, or relating to a whole group of similar things, rather than to any particular thing.
     ex) The new range of engines all had a generic problem with their fan blades.
> 2. generic drugs or other products do not have a trademark and are sold without the name of the company that produced them.


제네릭 사전 정의이다. 첫번째는 ‘특정 사물이 아닌 비슷한 사물 모두와 관련 있거나 전형적으로, 공식적으로 공유되는’ 을 뜻한다. 쉽게 말해서 ‘통칭해서/포괄적으로’ 를 뜻한다. 두번째 정의는 ‘상품을 생산 한 회사 이름없이 또는 일반 명사으로 판매되는’ 을 뜻한다. 노브랜드 또는 대일밴드, 봉고차, 노트북 등을 떠올리면 이해하기 쉽다. 방금 언급한 세 가지 제품은 특정 회사 제품 명이지만, 지금은 그냥 그 제품군 전체를 통칭해서 부르는 일반 명사이다.

## 자바에서 제네릭은 어떤 뜻으로 사용되는 것일까? 제네릭 클래스, 메서드, 인터페이스를 어떻게 이해하면 좋을까?

첫번째 사전 정의로 해석해서 포괄적으로 클래스, 통칭 메서드라는 단어의 의미를 생각해보자. 뭔가 이상하다. 클래스, 메서드는 이미 우리가 통칭해서(포괄적으로) 사용하는 단어이다. 따라서 자바에서 통칭한(포괄적인) 것은 클래스나 메서드가 아닌 타입이다. 즉, 특정 타입( JindoDog)이 아닌 그와 비슷한 타입(<T extends/super JindoDog>)이거나 전형적으로 또는 공식적으로 의미되는(Dog) 것 모두 사용가능한 클래스, 메서드로 이해할 수 있다.

두번째 사전 정의는 노브랜드다. 무엇이 노브랜드가 되는지 생각해보면 쉽다. 바로 클래스, 메서드가 다루는 타입이다. 타입을 특정하지 않고 일반 명사(타입)로써 클래스, 메서드를 만드는 것이다. 봉투는 어느 회사의 제품인지 구분하지 않고 봉투라고 부른다. 즉, 노브랜드 봉투가 있다. 여기에 과자를 담으면 과자 봉투고 과일을 담으면 과일 봉투다. 자바에 리스트가 있다. 스트링을 담으면 스트링 리스트이고 인티저를 담으면 인티저 리스트이다. 그러니까 노브랜드(어떤 타입인지 상관하지 않는) 리스트이다.

보통 첫번째 사전 정의로 많이 설명하고 있다. 두번째는 억지로 끼워 맞춘 감이 있지만, 따지고 보면 두 가지 정의의 의미가 비슷함을 알 수 있다. 이제까지 제네릭을 사용하면서 어렴풋하게 알고 있던 뜻을 사전 정의로 해석해봤다. 정리하면, 클래스(메서드, 인터페이스)를 선언할 때 사용하는 타입을 특정하지 않고 제네릭 타입으로 선언해서 특정 타입을 파라미터로 받아서 사용할 수 있게 한다. 이런 클래스, 메서드, 인터페이스를 제네릭 클래스, 제네릭 메서드, 제네릭 인터페이스라 한다. **즉, 타입을 파라미터화 한다.**

## 제네릭 기본 용법

앞선 글에서 클래스, 메서드 그리고 인터페이스까지 제네릭을 사용하는 것은 타입을 파라미터화해서 사용하는 것이라 했다. <T> 를 사용해서 제네릭 클래스, 메서드, 인터페이스를 만든다. 여기서 T 는 Type의 약자를 대문자로 표현한 것인데, 이것은 무엇이 되든 상관없다.

### 타입 파라미터의 관례적 명명
* E : Element (자바 컬렉션 프레임워크 소스를 보면 많이 사용되고 있다.)
* K : Key (Map 선언할 때, 자주 봤다.)
* N : Number
* T : Type (기본 값)
* V : Value (Map 선언할 때, 자주 봤다.)
* S, U, V etc. : 2nd, 3rd, 4th types (T 외에도 타입 파라미터가 필요할 때)

### Generic Class

제네릭 클래스 다음과 같은 의미를 담고 있다.

* * *

**이 클래스 인스턴스를 생성하려면 타입 아규먼트를 보내야 해! 전달된 아규먼트로 T를 대체해서 인스턴스가 생성 될거야!**

* * *

```java
public class Generics<T> {
    private T t;
    
    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}
```

### Generic Class Usage

```java
Generics<String> genercis = new Generics<>();
generics.set("Hello");
// list 사용할 때와 비교해보자. 똑같지 않나?
List<String> strs = new ArrayList<>();
```

### Generic Interface

```java
interface Generic<T1, T2> {
    T1 doSomething(T2 t);
    T2 doSomething2(T1 t);
}
```

### Generic Interface Implements

```java
// Comparable 인터페이스를 구현해 봤다면 알 것이다.
// public interface Comparable<T> {
//     public int compareTo(T o);
// }
class GenericInterfaceImpl implements Generic<String, Integer> {

    @Override
    public String doSomething(Integer t) {
        return t.toString();
    } 

    @Override
    public Integer doSomething2(String t) {
        return Integer.parseInt(t);
    }
}
```

제네릭 클래스와는 다르게 인스턴스 생성할 때, 타입 아규먼트를 보내는 것이 아니라 클래스에 인터페이스를 구현할 때 타입 아규먼트를 결정한다.

### Generic method

메서드 파라미터에 타입 파라미터도 선언되어 있다면, 메서드 리턴 타입 앞에 꼭 제네릭 타입이 선언되어야 한다.

```java
class GenericMethod {
    public static <T> int getListSize(List<T> list) {
        int count = 0;
        for (T t : list) {
            count++;
        }

        return count;
    }
}
```

어떤 타입의 리스트를 아규먼트로 보내도 동작한다.

```java
List<String> strs = Arrays.asList("a", "b", "c", "d");
List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5, 6);

int size1 = GenericMethod.getListSize(strs);
int size2 = GenericMethod.getListSize(ints);
```

## 제네릭을 사용하는 이유

* 런타임에서 오류를 발견하는 것보다 컴파일 타임에 타입 체크가 가능하게 코드를 작성할 수 있다. (타입 안정성 보장)
* 타입의 종류만 바꾸면 되는 로직일 경우, 코드 재활용이 가능하다. (코드 중복 제거)
  제네릭을 사용하지 않고 이 둘을 만족하는 코드는 작성이 불가능하다. 가능한 방법이 있다해도 효율성이 매우 낮을 것이다. 그렇다면 왜 불가능한지 알아보자.

### 타입 안정성 보장
타입 체크를 컴파일 타임에 할려면 타입을 특정해서 선언하면 된다.

```java
class AppleBox {
    
    Apple item;
    
    public void store(Apple item) {
        this.item = item;
    }
    
    public Apple getItem() {
        return item;
    }
}

class MelonBox {
    
    Melon item;
    
    public void store(Melon item) {
        this.item = item;
    }
    
    public Melon getItem() {
        return item;
    }
}
```

멜론 박스에 사과를 저장하려거나 사과 박스에 멜론을 저장하는 코드를 작성하면 컴파일 타임에 오류를 발견한다.

```java
AppleBox appleBox = new AppleBox();
appleBox.store(new Melon()); // 컴파일오류발생

MelonBox melonBox = new MelonBox():
melonBox.store(new Apple()); // 컴파일오류발생
```

지금까지 작성한 코드는 컴파일 타임에 오류를 발견하게 되어 타입 안정성을 보장한다. 그러나 코드 중복은 피할 수 없다.

### 코드 중복 제거
코드 중복 제거를 위해 Box 클래스의 타입을 Object로 바꿔보자.

```java
class ObjectBox {
    
    Object item;
    
    public void store(Object item) {
        this.item = item;
    }
    
    public Object getItem() {
        return item;
    }
}
```

Object 타입으로 변경하면서 하나의 클래스로 사과, 멜론 모두 처리가능하게 되었다. 그렇다면 타입 안정성은 컴파일 타임 체크가 가능할까? 개발자가 objectBox1 인스턴스에는 사과를 objectBox2 인스턴스에는 멜론을 넣고 나중에 꺼내 사용하는 코드를 작성하고 있다. 먼저 사과를 꺼내는 코드를 작성하고 그 코드를 복사해서 멜론을 꺼내는 코드로 수정했다. 그런데 실수로 objectBox2 를 objectBox1 으로 변경하는 작업을 빠트렸다. 어떻게 될까?

```java
ObjectBox objectBox1 = new ObjectBox();
objectBox1.store(new Melon()); // 컴파일통과

ObjectBox objectBox2 = new ObjectBox():
objectBox2.store(new Apple()); // 컴파일통과

Apple apple = (Apple)objectBox2.getItem(); // 컴파일통과
Melon melon = (Melon)objectBox2.getItem(); // 컴파일통과 런타임오류
```

개발자는 이를 컴파일 타임에 알지 못하고 런타임에 발견하게 된다. 즉, 타입 안정성을 보장 할 수 없다. 그래서 타입 안정성과 코드 중복 제거 모두를 할 수 있는 제네릭을 사용한다.

## Reference

* https://www.youtube.com/watch?v=ipT2XG1SHtQ