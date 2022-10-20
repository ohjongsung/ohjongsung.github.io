함수형 인터페이스는 오직 하나의 추상 메소드만 갖는 인터페이스를 말한다. Java 8 부터 가능해진 디폴트 메서드는 여러개 있어도 추상 메소드만 하나면, 함수형 인터페이스이다. 함수형 인터페이스를 사용하는 이유는 자바의 람다식은 함수형 인터페이스로만 사용이 가능하기 때문이다.

```java
// @FunctionalInterface 어노테이션은 해당 인터페이스가 함수형 인터페이스가 맞는지 검사
@FunctionalInterface 
public interface FunctionalInterface {
    public abstract void doSomething(String text);
		
		// default method 는 존재해도 상관없음
    default void printDefault() {
        System.out.println("Hello Default");
    }

    // static method 는 존재해도 상관없음
    static void printStatic() {
        System.out.println("Hello Static");
    }
}
```

## 자바 기본 함수형 인터페이스
[ java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html) 을 보면, 더 많은 함수형 인터페이스를 확인할 수 있다.


|제목|내용|설명|
|:------|:---|:---|
|Predicate|T -> boolean|boolean test(T t)|
|Consumer|T -> void|void accept(T t)|
|Supplier|() -> T|T get()|
|Function<T, R>|T -> R|R apply(T t)|
|Comparator|(T, T) -> int|int compare(T o1, T o2)|
|Runnable|() -> void|void run()|

## Predicate

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

## Consumer

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

## Supplier

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

## Function

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

## Comparator

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

## Runnable

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
