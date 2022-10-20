Optional default value 를 사용할 때, 헷갈리는 orElse 와 orElseGet 의 차이점을 알아보자. 두 기능은 서로 매우 비슷해보이지만, 내부적으로 큰 차이점이 있다. 자바 문서를 보면 다음과 같다.

## orElseGet

```java
    /**
     * Return the value if present, otherwise invoke {@code other} and return
     * the result of that invocation.
     *
     * @param other a {@code Supplier} whose result is returned if no value
     * is present
     * @return the value if present otherwise the result of {@code other.get()}
     * @throws NullPointerException if value is not present and {@code other} is
     * null
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

## orElse

```java
    /**
     * Return the value if present, otherwise return {@code other}.
     *
     * @param other the value to be returned if there is no value present, may
     * be null
     * @return the value, if present, otherwise {@code other}
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

orElseGet 에는 Supplier 가 parameter 로 전달된다. orElse 에서는 그냥 objet T 가 전달된다. 이 차이점은 실제 어떻게 동작하게 되는 것일까? 아래와 같은 소스를 실행해보자.

```java
    public static void main(String[] args) {
        System.out.println("empty optional");
        Optional.empty().orElse(getDefaultValue());
        Optional.empty().orElseGet(OptionalGet::getDefaultValue);
    }

    public static String getDefaultValue() {
        System.out.println("getDefaultValue");
        return "default value";
    }
```

결과

```text
empty optional
getDefaultValue
getDefaultValue
```

이번에는 실제로 값이 있는 Optional 로 실행해보자.

```java
    public static void main(String[] args) {
        System.out.println("hello world");
        Optional<String> optional = Optional.of("hello world");
        optional.orElse(getDefaultValue());
        optional.orElseGet(OptionalGet::getDefaultValue);
    }

    public static String getDefaultValue() {
        System.out.println("getDefaultValue");
        return "default value";
    }
```

결과

```text
hello world
getDefaultValue
```

앞선 두 코드의 실행결과를 보면, orElseGet 은 비어있는 Optional 객체가 넘어온 경우에만 Supplier 함수형 인자를 사용해서 생성된 값을 반환한다. 그러나 orElse 는 비어있는 Optional 객체가 넘어온 경우 Object 객체를 반환하지만, Optional 객체의 값 존재 여부와 상관없이 Object 객체를 생성한다.

## 마무리

orElseGet 은 Optional 객체가 비어있는 경우에만 함수가 호출되기 때문에 orElse 보다 성능상 이점이 있다.

## Reference

* http://www.daleseo.com/java8-optional-after/
* https://fedotjava.wordpress.com/2017/01/29/java-8-whats-the-difference-between-the-optional-orelse-and-orelseget/