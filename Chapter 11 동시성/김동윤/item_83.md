# 지연 초기화는 신중히 사용하라
## 지연 초기화(lazy initialization)
- 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
- 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는다.
- 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과 ㅇㅇ
- 다른 모든 최적화와 마찬가지로 지연 초기화는 "필요할 때까지는 하지말라".
### 최적화에서의 지연초기화
- 클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄지만,
지연 초기화하는 필드에 접근하는 비용이 커진다.
- 지연 초기화 필드중 결국 초기화가 이뤄지는 비율, 시렞 초기화에 드는 비용, 초기화된 각 필드의 호출 빈도에 따라 성능 판단

### 지연 초기화가 필요한 시점
- 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮고, 그 필드를 초기화하는 비용이 클 때 (hashCode)
- 지연 초기화 적용 전후의 성능을 측정

### 멀티 스레드 환경에서 지연 초기화
- 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화
- 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다. 

## 지연 초기화 방법 
### 일반적인 인스턴스 필드 초기화 
대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

```java
private final FieldType field1 = computeFieldValue();
```

### 인스턴스 필드의 지연초기화
- 지연 초기화가 초기화 순환성(initialization circularity)을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자.

```java
private FieldType field2;
private synchronized FieldType getField2() {
  if (field2 == null)
    field2 = computeFieldValue();
  return field2;
}
 
```
### 정적 필드용 지연 초기화 홀더 클래스 관용구
```java
private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

private static FieldType getField() { 
	return FieldHolder.field; 
}
```
- 성능때문에 정적 필드를 지연초기화해야 한다면 지연초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자
- 클래스는 클래스가 처음 쓰일 때 비로소 초기화 된다는 특성이 있다.

### 짜릿한 단일검사(racy single-check) 관용구
```java
private FieldType field5;

private FieldType getField5() {
  FieldType result = field5;
  if (result == null)
    field5 = result = computeFieldValue();
  return result;
}

private static FieldType computeFieldValue() {
  return new FieldType();
}
```
- 모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사의 필드선언에서 volatile을 없애도 됨~~~

## (+) volatile 추가 조사
https://nesoy.github.io/articles/2018-06/Java-volatile
- volatile keyword는 Java 변수를 Main Memory에 저장하겠다라는 것을 명시하는 것 
- 매번 변수의 값을 Read할 때마다 CPU cache에 저장된 값이 아닌 Main Memory에서 읽는 것 
- 또한 변수의 값을 Write할 때마다 Main Memory에 까지 작성하는 것

=> 으음 ~ 그러니깐 보통의 변수들은 처음엔 메인 메모리, 그 이후에는 캐시에서 변수를 읽어오는게 일반적이야, 그런데 캐시에서만 읽어오면 이 변수가 메인메모리에서 변경되었더라도!! 캐시에 저장된 값으로만 읽혀져오는 불상사가 일어나게 되겠지!

![](https://velog.velcdn.com/images/myway00/post/807f205d-e3f0-46cf-b808-39bbb4911c3a/image.png)


> 만약에 Multi Thread환경에서 Thread가 변수 값을 읽어올 때 각각의 CPU Cache에 저장된 값이 다르기 때문에 변수 값 불일치 문제가 발생하게 되는 것!!!


### 어떻게(How) 해결할까요?
- volatile 키워드를 추가하게 되면 Main Memory에 저장하고 읽어오기 때문에 변수 값 불일치 문제를 해결

```java
public class SharedObject {
    public volatile int counter = 0;
}
```

### 언제(When) volatile이 적합할까요?
- Multi Thread 환경에서 하나의 Thread만 read & write하고 나머지 Thread가 read하는 상황에서 가장 최신의 값을 보장

### volatile 성능에 어떤 영향이 있을까요?
- volatile는 변수의 read와 write를 Main Memory에서 진행 
- CPU Cache보다 Main Memory가 비용이 더 크기 때문에 변수 값 일치을 보장해야 하는 경우에만 volatile 사용이 better
### volatile 결론 
- Main Memory에 read & write를 보장하는 키워드.
#### 사용 상황?
- 하나의 Thread가 write하고 나머지 Thread가 읽는 상황인 경우.
- 변수의 값이 최신의 값으로 읽어와야 하는 경우.
#### 주의할 점?
- 성능에 영향이 어느 정도 영향을 줄 수 있는 Point라는 점.

## REFERENCE 
https://devfunny.tistory.com/857?category=895441
https://javabom.tistory.com/92?category=833277