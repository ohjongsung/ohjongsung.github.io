## Java Container Heap Memory 설정 변화 흐름

### JDK 8u131+, JDK 9
컨테이너에 할당된 메모리의 25% 를 힙 메모리로 할당한다.
```
-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap
```
MaxRAMFraction 설정으로 비율을 조정할 수 있다.

1 = 할당 메모리 100%

2 = 할당 메모리 50%

3 = 할당 메모리 33%

4 = 할당 메모리 25% 를 힙 메모리로 할당한다. (기본값)
```
-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=2
```

### JDK 8u191+, JDK 10, 11 etc
UseContainerSupport 옵션이 기본적으로 활성화 되었기 때문에 컨테이너에서 설정한 메모리를 자동으로 인식하고 적용된다.

(JDK 10 에 추가된 기능을 JDK 8u191+ 에도 적용해줌)

즉, 아무런 설정을 하지 않아도 자동으로 25% 비율로 힙 메모리를 할당한다.

그리고 동일하게 MaxRAMFraction 으로 힙 메모리 비율을 조정 할 수 있다.
```
-XX:MaxRAMFraction=2
```

### JDK 10+
**InitialRAMPercentage, MinRAMPercentage, MaxRAMPercentage**
MaxRAMFraction 설정보다 커스텀하기 쉬워서 메모리 낭비를 줄일 수 있는 설정

(JDK 10 에 추가된 기능을 JDK 8u191+ 에서도 사용가능하게 적용해 줌)
```
-XX:InitialRAMPercentage=75.0 -XX:MinRAMPercentage=50.0 -XX:MaxRAMPercentage=75.0
```

#### InitialRAMPercentage
'-XX:InitialRAMPercentage' JVM 인수는 자바 애플리케이션의 초기 힙 크기를 계산하는 데 사용된다.

-XX:InitialRAMPercentage=25.0 로 설정하고 전체 물리적 메모리(또는 컨테이너 메모리)가 1GB인 경우 Java 애플리케이션의 힙 크기는 ~250MB(즉, 1GB의 25%)가 된다.

#### MinRAMPercentage
'-XX:MinRAMPercentage' JVM 인수는 물리적 서버(또는 컨테이너)에서 사용 가능한 전체 메모리 크기가 **250MB(대략) 미만인 경우에만 Java 힙 크기를 계산하는 데 사용된다.**

-XX:MinRAMPercentage=50.0 로 설정하고 전체 물리적 메모리(또는 컨테이너) 메모리가 100MB라고 가정하면 Java 애플리케이션의 최대 힙 크기는 50MB(즉, 100MB의 50%)로 설정됩니다.

JDK 개발팀이 이름을 엉망으로 지어서, 최대 힙 사이즈를 설정하는 값으로 착각하게 만든다.

**프로덕션 레벨에서 전체 메모리 크기를 250MB 이하로 할 일이 거의 없으므로 사용할 일이 없어 보인다.**

#### MaxRAMPercentage
‘-XX : MaxRamperCentage’ JVM 인수는 실제 서버 (또는 컨테이너)에서 전반적인 메모리 크기가 **250MB (대략) 이상인 경우에만 Java 힙 크기를 계산하는 데 사용된다.**

-XX : MaxRamperCentage = 75.0 로 설정하고 전체 물리 서버 (또는 컨테이너) 메모리가 1GB라고 가정하면 Java 응용 프로그램의 최대 힙 크기는 750MB (즉, 1GB의 75%)로 설정된다.



### 주의사항
'-XX:InitialRAMPercentage'는 '-Xms' JVM 인수가 전달되지 않은 경우에만 초기 힙 크기를 설정하는 데 사용된다.

'-Xms' JVM 인수가 전달되면 JVM에서 '-XX:InitialRAMPercentage'를 무시한다.

'-XX:MaxRAMPercentage' 및 '-XX:MinRAMPercentage'는 '-Xmx' JVM 인수가 전달되지 않은 경우에만 최대 힙 크기를 설정하는 데 사용된다.

'-Xmx' JVM 인수가 전달되면 JVM은 이 두 인수를 무시한다.

## Reference

* https://dzone.com/articles/best-practices-java-memory-arguments-for-container
* https://jogeum.net/32
* https://dzone.com/articles/difference-between-initialrampercentage-minramperc