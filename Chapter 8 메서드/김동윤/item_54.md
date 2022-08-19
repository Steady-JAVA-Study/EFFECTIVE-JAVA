# null이 아닌, 빈 컬렉션이나 배열을 반환하라

## null 처리
```java

public class Main {
    private final static List<Cheese> cheesesInStock = new ArrayList<>();
 
    /**
     * 재고가 없다면 null 반환
     * @return
     */
    public static List<Cheese> getCheeses() {
        return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
    }
 
    public static void main(String[] args) {
       
    }
 
    enum Cheese {
        STILTON
    }
}
```
- 재고가 없다면 null이 반환
- 클라이언트는 이 null 상황을 처리하는 코드를 추가로 작성

## 해결방안 1. 빈 리스트 반환
- null이 아닌 빈 배열이나 빈 컬렉션을 반환
- 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환 가능

```java
//길이가 0일수도 있는 배열을 반환하는 방법
public Cheese[] getCheeses() {
   return cheesesInStock.toArray(new Cheese[0]);
}
```


## 해결방안 2. 빈 배열 반환
- 매번 새로 할당하는 위 방식이 성능을 떨어트릴 것 같다면, 미리 길이 0짜리 배열을 선언해두고 매번 그것을 반환

```java
//최적화 - 빈 배열을 매번 새로 할당하지 않도록 함.
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
   return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```


## 결론
- null이 아닌, 빈 배열이나 컬렉션을 반환 
- null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다 
