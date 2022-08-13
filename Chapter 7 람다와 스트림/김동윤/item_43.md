# 람다보다는 메서드 참조를 사용하라
- 람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결
- BUT 자바에서는 함수 객체를 심지어 람다보다도 더 간결하게 만드는 방법이 있으니, 바로 **메서드 참조**

>  - 람다 표현식이 단 하나의 메소드만을 호출하는 경우에 해당 람다 표현식에서 불필요한 매개변수를 제거하고 사용 가능케 해줌
- 세미콜론 두개(::)로 메소드를 참조

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

여기서 변수가 중복이 된다

```java
map.merge(key, 1, Integer::sum)
```

## 메서드 참조의 유형 다섯 가지

### 1. 정적
Integer::parseInt
str -> Integer.parseInt(str)


### 2. 한정적(인스턴스)
Instance.now()::isAfter
Instance then = Instant.now(); t -> then.isAfter(t)


### 3. 비한정적(인스턴스)
String::toLowerCase
str -> str.toLowerCase()


### 4. 클래스 생성자
TreeMap<K, V>::new
() -> new TreeMap<K, V>()


### 5. 배열 생성자
int[]::new
len -> new int[len]

## 결론
> - 메서드 참조는 람다의 간단명료한 대안이 될 수 있음
- 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용!
_____________________________________ 

### 출처
https://hirlawldo.tistory.com/97