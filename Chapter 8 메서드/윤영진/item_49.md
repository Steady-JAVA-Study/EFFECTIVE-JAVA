# ITEM 49. 매개변수가 유효한지 검사하라

메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건(불변식)을 만족했을 때 제대로 동작해야 한다. 그리고 이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다.

## 매개변수 검사하는 위치는 반드시 메서드 시작 부분을 권장한다.

1. 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
2. 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
3. 불필요한 호출이 일어나거나 테이블에 데이터를 넣는 도중에 에러가 발생하는 경우가 일어날 수 있다.

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.(@throws 자바독 태그를 사용하면 된다) 자바 7에 추가된 java.util.Objects.requireNonNull 메서드는 유연하고 사용하기도 편하니, 더 이상 null 검사를 수동으로 하지 않아도 된다.

> Objects.requireNonNull이란?
> 
> 특정한 객체 참조가 널인지 판단하고, null인 경우 사용자 지정 NullPointException을 발생시킨다. 이 메소드는 주로 메서드나 생성자의 매개변수 유효성 검증을 위해 설계 되었다.

```text
this.strategy = Objects.requiredNonNull(strategy, "전략");
```

자바 9에서는 Objects에 범위 검사 기능도 더해졌다. checkFromIndexSize, chechFromToIndex, checkIndex라는 메서드들인데, null 검사 메서드만큼 유연하지는 않다.

## public 메서드가 아니라면 단언문(assert)를 사용할 수 있다.

공개되지 않은 메서드는 개발자가 메서드 호출을 통제할 수 있다. 따라서 매개변수 유효성을 검증하고 오직 유효한 값만이 메서드에 넘겨진다는 것을 보증해야 한다.

단언문 assert를 통해 실행되는 문장이 참이라는 것을 보증할 수 있다.

```java
private static void sort(long a[]. int offset, int length){
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // 계산 수행
}
```

assert는 Java4부터 지원하며 자신이 단언한 조건이 무조건 참이라고 선언한다. 즉, 실행되는 문장이 참(true)이라면 그냥 지나가고, 거짓(false)이라면 AssertionError가 발생한다.

---

1. MVC 모델에서 입력에 대한 검증은 Controller, Service 레이어에서 처리
2. 비즈니스 validate(userId 검사, ...) Service 레이어에서 처리
   1. 매개변수 검사는 순서는 변화가능성이 적은 것부터 수행하는게 좋다.

