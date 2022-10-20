Java Collection에 대해서 공부하다 보면, 초기값을 명시하지 않고 Default 생성자로 ArrayList를 생성 시 내부적으로 크기가 10인 Object 배열을 생성한다는 글을 많이 봤다. 마찬가지로 HashMap도 Default 생성자로 생성 시 Capacity가 16인 Map을 생성한다고 했다. 그런데 JDK 1.7.0_40 update에서 요게 바뀌었단다.

## java.util.ArrayList

#### JDK 1.6.30

```java
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this(10);
    }
 
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
        this.elementData = new Object[initialCapacity];
    }
```

Default 생성자로 ArrayList를 생성 시 내부적으로 크기가 10인 Object 배열을 생성하고 있다. 이는 데이터를 추가하지 않더라도 항상 크기 10인 Object가 메모리를 차지하고 있음을 뜻한다.

#### JDK 1.7.0_40

```java
    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
 
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }
```

빈 Object 배열을 정적 변수에 선언해서 Default 생성자로 ArrayList를 생성하면 해당 빈 Object를 초기값으로 설정하게 되었다.

## java.util.HashMap

#### JDK 1.6.30

```java
    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        threshold = (int) (DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
        table = new Entry[DEFAULT_INITIAL_CAPACITY];
        init();
    }
```

HashMap은 Default 생성자로 생성 시 Capacity가 16인 Map을 생성하고 있었다.

#### JDK 1.7.0_40

```java
    /**
      * An empty table instance to share when the table is not inflated.
      */
    static final Entry<?, ?>[] EMPTY_TABLE = {};
 
    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     */
    transient Entry<K, V>[] table = (Entry<K, V>[]) EMPTY_TABLE;
 
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
    // .....
    }
```

ArrayList와 비슷하게 빈 Entry 배열을 정적 변수에 선언해서 사용하게 변경되었다.

## 마무리

Java 업데이트가 있을 때마다 릴리즈 노트를 꼼꼼히 읽지 않는 한 알 수가 없지 않나 싶지만, ArrayList와 HashMap은 정말 많이 사용하는 Collection이니 알아두는 게 좋지 않을까? 물론 너무 옛날이긴 하다. 아무튼 업데이트 이전의 방식은 쓸데없는 메모리를 차지하고 있었다. 이는 Garbage Collection에도 분명 영향을 끼쳤을 것이다. 그래서 별거 아닌 업데이트 같지만 성능 향상에 큰 도움이 되었을 것이다.

## Reference
* <http://javarevisited.blogspot.kr/2014/07/java-optimization-empty-arraylist-and-Hashmap-cost-less-memory-jdk-17040-update.html>