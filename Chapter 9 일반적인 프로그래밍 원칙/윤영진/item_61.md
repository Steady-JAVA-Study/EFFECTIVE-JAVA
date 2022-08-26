# ITEM 61. 박싱된 기본 타입보다는 기본 타입을 사용해라

## 기본 타입과 박싱된 기본 타입의 차이

1. 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값 + 식별성이라는 두 가지 속성을 갖는다.
    - 다시 말하면, 값이 같은 박싱된 기본 타입의 인스턴스가 두 개 존재할 때, 이 두 개는 서로 다르다고 식별될 수 있다.

2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 값을 가질 수 있다. 즉, null을 가질 수 있다.
3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

### 박싱된 기본 타입은 값 + 식별성이라는 속성을 갖는다.

```java
public static void main(String [] args) {
  Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
  int num = naturalOrder.compare(new Integer(42), new Integer(42));
  System.out.println(num);
}
//실행 결과 : 1
```

- 처음 (i < j) 의 검사는 잘 작동한다. 여기에서 i와 j가 참조하는 오토박싱된 Integer 인스턴스는 기본 타입으로 변환된다.
- 즉, 첫번째 정수값(i)이 두번째 정수값(j)보다 작은지 확인한다.
- 그리고 첫번째 정수값(i)이 두번째 정수값(j)보다 작지 않으면 두번째 검사인(i == j)가 이루어진다. 그런데 이 두번째 검사에서는 두 객체 참조의 식별성 검사를 하게 된다.
  - 즉, i와 j가 서로 다른 Integer 인스턴스라면 비교의 결과는 false가 되고 비교자는 1을 반환하게 된다.

### 박싱된 기본 타입(int가 Integer로 박싱)은 null을 가질 수 없다.

```java
public class BoxingNull {
  static Integer i;

  public static void main(String [] args) {
    if(i == 43) {
      System.out.println("믿을 수 없네..!");
    }
  }
}
```

위 소스의 결과를 확인하면 NPE가 발생하는 것을 확인할 수 있다.

원인은 i가 int(기본타입)이 아닌 Integer(참조타입)이기 때문이다. 즉, i==43은 Integer와 int를 비교하는 것이다.

1. 거의 예외없이 기본 타입과 박싱된 기본타입(int가 Integer로 박싱)을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다.
2. 다시 말해, 기본 타입으로 다시 돌아간다는 얘기다.
3. 그로 인해 null 참조를 언박싱(기본 타입으로 돌아감)하면 NullPointerException이 발생한다.


### 3. 기본타입이 박싱된 기본타입보다 시간과 메모리 사용면에 더 효율적이다.

```java
Long sum = 0L;
for(long i = 0; i <= Integer.MAX_VALUE; i++) {
  sum += i;
}
System.out.println(sum);
```
위 소스는 지역변수 sum을 박싱된 기본타입인 Long으로 선언하여 오류없이 컴파일은 되지만, 박싱과 언박싱으로 반복해서 일어나므로 성능상 매우 느려진다.

### 박싱된 기본타입 사용해야하는 경우

1. 컬렉션의 원소, 키, 값으로 쓸 때
2. 리플렉션을 통해 메서드를 호출할 때
3. null 값 처리가 용이하기 때문에 SQL과 연동할 경우에 처리를 원활하게 할 수 있다.
   - DB에서 자료형이 정수형이지만 null 값이 필요한 경우 VO에서 Integer를 사용할 수 있다.
