# ITEM 75. 예외의 상세 메시지에 실패 관련 정보를 담으라

예외를 잡지못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적(stack trace) 정보를 자동으로 출력한다. 스택 추적은 예외 객체의 `toString()` 메서드를 호출해서 얻는 문자열로, 보통은 예외의 클래스 이름 뒤에 상세 메시지가 붙는 형태다.

따라서 예외의 `toString()` 메서드에 실패 원인에 관한 정보를 가능한 한 많이 담아 반환하는 일은 아주 중요하다. 달리 말하면, 사후 분석을 위해 실패 순간의 상황을 정확히 포착해 예외의 상세 메시지에 담아야 한다.

```java
public class IndexOutOfBoundsException extends RuntimeException {

    private final int lowerBound;
    private final int upperBound;
    private final int index;
    
    ...

    public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {

        super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));

        this.lowerBound = lowerBound;
        this.upperBound = upperBound;
        this.index = index;
    }
}
```


- 포착한 실패 정보는 예외 상황을 복구하는데 유용할 수 있으므로 접근자 메서드는 uncheckedException 보다는 checkedException 에서 더 빛을 발한다. 
- 하지만 'toString이 반환 값에 포함된 정보를 얻어올 수 있는 API 를 제공하자' 는 일반원칙을 따른다는 관점에서 비검사 예외라도 상세 정보를 알려주는 접근자 메서드를 제공하라고 권하고 싶다.

