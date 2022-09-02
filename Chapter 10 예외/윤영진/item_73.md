# ITEM 73. 추상화 수준에 맞는 예외를 던지라 



## 예외 번역

상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.

```java
try {
    ...// 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    // 추상화 수준에 맞게 번역한다. 
    throw new HigherLevelException(...);
}
```

### 예외 인쇄

예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 **예외 연쇄**를 사용하는게 좋다

문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식

`Throwable.getCause()`를 통해 언제든 저수준 예외를 꺼내볼 수 있다

```java
try {
  ...
} catch (LowerLevelException cause) {
  throw new HigherLevelException(cause);
}
```
```java
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

**무턱대고 예외를 전파하는 것보다야 예외 번역이 우수한 방법이지만, 그렇다고 남용해서는 곤란하다.**

가능하다면 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.


**아래 계층에서 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 않는 방법이 있다.**

```java
public void function1(...) {
  ...
  try {
    function2(...);
  } catch(...) {
    //로그 기록
  }
}
```
이 경우 발생한 예외는 java.util.logging 같은 적절한 로깅 기능을 활용하여 기록해두면 좋다.

클라이언트 코드와 사용자에게 문제를 전파하지 않으면서도 프로그래머가 로그를 분석해 추가 조치를 취할 수 있게 해준다.
