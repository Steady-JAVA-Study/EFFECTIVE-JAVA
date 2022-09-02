# ITEM 74. 메서드가 던지는 모든 예외를 문서화하라

메서드가 던지는 예외는 그 메서드를 올바르게 사용하기 위한 중요한 정보이므로 예외 하나하나를 문서화하는데 충분한 시간을 쏟아야한다.

- 검사 예외는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독의 `@throws` 태그를 사용하여 정확히 문서화하자.
- 공통 상위 클래스 하나로 예외를 던지는 것을 삼가자 ex)`Exception` `Throwable`
- 비검사 예외도 검사 예외처럼 정성껏 문서화해두면 좋다.
- 메서드가 던질 수 있는 예외를 `@throws` 로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.

```java
/**
 * ...
 * @throws SQLExceptio SQL 이 잘못된 경우
 * @throws ClassNotFoundException 지정한 경로에 클래스파일이 존재하지 않는 경우 
 */
public void ex() throws SQLException, ClassNotFoundException {
    
        ...
}
```
