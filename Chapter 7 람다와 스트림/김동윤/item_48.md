## 스트림 동시성 vs 스트림 병렬화
- 둘 다 멀티 스레드의 동작 방식이라는 점은 동일하지만 목적이 다름 !

![](https://velog.velcdn.com/images/myway00/post/8bbe1194-c4af-46bc-ae4a-da37bdc70a02/image.png)

### 동시성(Concurrency)
- 멀티 작업을 위해 멀티 스레드가 번갈아가며 실행하는 성질

### 병렬성(Parallelism)
- 병렬성은 멀티 작업을 위해 멀티 코어를 이용해 동시에 실행하는 성질

## 병렬 스트림 ? 
> 병렬 스트림은 병렬화하기 위해 스트림을 재귀적으로 분할하고, 스레드를 할당하고, 최종적으로 부분 결과를 하나로 합치는 과정
- 병렬 스트림은 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림
- So, 병렬 스트림을 이용하면 멀티 코어 프로세서가 각각의 청크를 처리하도록 할당 가능
- 병렬스트림 내부에서 포크/조인 프레임워크를 활용

![image](https://user-images.githubusercontent.com/76711238/184495816-3ae4f940-5126-4a3c-800a-f4fcc29d234a.png)
![image](https://user-images.githubusercontent.com/76711238/184495834-80ccbd13-aaaf-42cf-b755-ac28e5895d96.png)
그림 출처 : https://wonyong-jang.github.io/java/2021/02/07/Java-Stream-parallel.html

# 스트림 병렬화는 주의해서 적용하라
- Java8부터는 parallel 메서드만 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원
- 동시성 프로그램을 작성하기는 쉬워지고 있지만, 올바르고 빠르게 작성하는 일은 여전히 어려운 일
- 동시성 프로그래밍을 할 때는 안전성(Safety)과 응답가능(liveness) 상태를 유지하기 위해 애써야 하는데 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없음

```
-Java5

java.util.concurrent 라이브러리 지원
Executor 프레임워크 지원

- Java7

고성능 병렬분해 프레임워크인 fork-join 추가

- Java8

병렬 스트림 실행 parallel메서드

```

### 파이프라인 병렬화로 성능 개선을 기대하기 어려운 경우
```java
// 스트림을 사용해 처음 20개의 메르센 소수를 생성하는 프로그램
package package2;

import java.math.BigInteger;
import java.util.stream.Stream;

public class Phase {


	    public static void main(String[] args) {
	        primes()//.parallel()
            .map(p -> BigInteger.TWO.pow(p.intValueExact()).subtract(BigInteger.ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
}

		public static Stream<BigInteger> primes() {
		    return Stream.iterate(BigInteger.TWO, BigInteger::nextProbablePrime);
		}

}
```

### 이 프로그램의 성능을 올리는 방법으로 병렬 처리를 떠올려서, parallel()을 호출해 병렬 처리하면 어떻게 되냐?

```java
package package2;

import java.math.BigInteger;
import java.util.stream.Stream;

public class Phase {


	    public static void main(String[] args) {
	        primes().parallel() //parallel 추가 ~ !
            .map(p -> BigInteger.TWO.pow(p.intValueExact()).subtract(BigInteger.ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20) // 처음 20개 선택 
            .forEach(System.out::println);
}

		public static Stream<BigInteger> primes() {
		    return Stream.iterate(BigInteger.TWO, BigInteger::nextProbablePrime);
		}

}
```
### 안되는 이유 : 
#### 1. 데이터 소스가 Stream.iterate인 경우

- parallel은 병렬로 실행하기 위해서 데이터 소스를 chunk 단위로 자르는데,
- iterate는 순차적으로 다음 요소를 반환하는 방식이라 chunk 단위로 자르기 어렵

#### 2. 중간 연산으로 limit() or findFirst 같이 요소의 순서에 의존하는 연산을 사용하는 경우
- 매 소수마다 새로이 계산을 하다보니 성능이 끔찍~
- limit 대신 rangeClosed를 쓰자 (LongStream.rangeClosed(1,n))
- limit 같은 것은 소수 찾기처럼 나중에 갈수록 오래걸리는 케이스에 더 주의 !

##  병렬화의 효과가 좋을 때?
### 1. 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스이거나 배열의 int 범위, long 범위일 때 병렬화의 효과가 가장 good
**1) 데이터의 분할이 정확하고 쉽다**

- 일을 다수의 스레드에 분배하기 좋다
- Spliterator를 이용해 데이터를 분할할 수 있다

**2) 참조 지역성이 뛰어나다**

- 참조 지역성이 낮으면 캐시 실패 확률이 높아지고, 이에 따라 스레드가 낭비 
- 따라서 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용 
- 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열 

### 2. 종단 연산이 병렬화에 적합할 때

- 축소(파이프라인에서 만들어진 모든 원소를 합치는 작업) 연산이 가장 병렬화에 적합하다
- Stream의 reduce 메서드 중 하나, 혹은 min, max, count, sum 같이 완성된 형태로 제공되는 메서드 중 하나
   - reduce ? 
   - reduce() 메소드는 첫 번째와 두 번째 요소를 가지고 연산을 수행한 뒤, 그 결과와 세 번째 요소를 가지고 또다시 연산을 수행
- anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드
- 가변 축소(mutable reduction)를 수행하는 Stream의 collect 메서드는 병렬화에 적합 X
- 컬렉션들을 합치는 부담이 크기 때문



## 출처
https://girawhale.tistory.com/131 (병렬 스트림에 관하여)
https://donghyeon.dev
https://cotak.tistory.com/211
https://www.baeldung.com/java-when-to-use-parallel-stream
