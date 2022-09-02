#  메서드가 던지는 모든 예외를 문서화하라

- 메서드가 던지는 예외는 그 메서드를 사용하는데 아주 중요한 정보다
- 따라서 각 메서드가 던지는 예외를 문서화하는 것은 중요하며 충분한 시간을 쏟아야 하지

## 검사 예외는 따로따로 선언하자
- 검사 예외는 항상 따로따로 선언하고, 
- 각 예외가 발생하는 상황을 자바독의 @throws 태그를 활용해 정확히 문서화 하자. 
- 공통 상위 클래스 하나로 통합해서 선언하는 일은 삼가라

```java
// 잘못 선언한 예
public void testMethod() throws Exception {

}

// or

public void testMethod() throws Throwable {

}
```

(ex) 위와 같이 Exception이나 Throwable을 던진다고 선언해서는 x
- 메서드 사용자에게 각 예외에 대처할 수 있는 힌트를 주지 x 
- 같은 맥락에서 발생할 수 있는 다른 예외들까지 모두 삼켜버려 API 사용성을 크게 떨어트린다.
- 다만, main 메서드는 오직 JVM만이 호출하므로 Exception을 던지도록 선언해도 ㄱㅊ
- 올바르게 사용하면 아래와 같음

```java
/**
 * @throws IllegalStateException
 */
public void testMethod(String parameter) throws IllegalStateException {
  
}
```

## 비검사 예외도 문서화하자
- 자바에서 요구하는 것은 아니지만 비검사 예외도 검사 예외처럼 문서화해두면 좋다. 
- 비검사 예외는 일반적으로 프로그래밍 오류
- 따라서 자신이 일으킬 수 있는 오류들이 무엇인지 알려주면 프로그래머는 자연스럽게 오류가 발생하지 않게 코딩 

## 한 클래스의 많은 메서드가 같은 이유로 같은 예외를 던진다면 각각에 메서드에 선언하기보다 클래스 설명에 추가하는 방법을 고려하자.
```java
/**
 * @throws NullPointerException - 
 모든 메서드는 param에 null이 넘어오면 
 NullPointerExcetpion을 던진다.
 */
class Dummy throws NullPointerException {

   public void methodA(String param) {
       ...
   }

   public void methodB(String param) {
       ...
   }

   public void methodC(String param) {
       ...
   }
}
```

## 결론
- 메서드가 던질 가능성이 있는 모든 예외를 문서화 
- 검사 예외든, 추상 메서드든 구현 메서드든 모두 문서화 
- 예외를 문서화할 때는 @throws 태그를 사용 
- 검사 예외만 throws 문에 일일이 선언하고, 비검사 예외는 메서드 선언에 기입 X

### reference 
https://jjingho.tistory.com/123