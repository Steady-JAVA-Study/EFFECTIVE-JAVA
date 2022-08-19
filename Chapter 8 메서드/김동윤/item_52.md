# 다중정의는 신중히 사용하라
- 다중정의(Overloading)는 어느 메서드가 호출될 지 컴파일 타임에 정해짐 


## 다중 정의
```java
import java.math.BigInteger;
import java.util.*;
 
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }
 
    public static String classify(List<?> lst) {
        return "리스트";
    }
 
    public static String classify(Collection<?> c) {
        return "그 외";
    }
 
    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };
 
        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```
```
예상 : "집합", "리스트", "그 외"
실제 결과 : "그 외", "그 외", "그 외"
```
- classify 메서드는 다중정의
- 위의 결과가 나온 이유는 다중정의(overloading)된 총 3개의 classify 중 어느 메서드를 호출해야할지가 컴파일 타임에 정해지기 때문
- 컴파일 타임에는 for문 안의 c는 항상 Collection<?> 타입

## 메서드 재정의
- 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택
-  메서드를 재정의했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준

```java
import java.util.List;
 
class Wine {
    String name() {
        return "포도주";
    }
}
 
class SparklingWine extends Wine {
    @Override String name() {
        return "발포성 포도주";
    }
}
 
class Champagne extends SparklingWine {
    @Override String name() {
        return "샴페인";
    }
}
 
public class OverridingTest {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());
 
        for (Wine wine : wineList)
            System.out.println(wine.name());
    }
}
```
```
포도주
발포성 포도주
샴페인
```

- 메서드를 재정의한 후, '하위 클래스의 인스턴스'에서 그 메서드를 호출하면 재정의한 메서드가 호출
- for 문에서의 컴파일 타임 타입이 모두 Wine인 것에 무관하게 항상 '가장 하위에서 정의한' 재정의 메서드가 실행되는 것

## 다양한 메서드 이름으로 다중정의 사용 지양
- 프로그래머에게는 재정의가 정상적인 동작 방식이고, 다중 정의가 예외적인 동작으로 보일 것이다. - 다중정의한 메서드는 우리의 예상을 빗나갔다. 헷갈릴 수 있는 코드는 작성하지 않는게 좋다. 
- 특히나 공개 public API라면 더더욱 신경 써야 한다. 다중정의가 혼동을 일으키는 상황을 피해야한다. 

## reference
https://devfunny.tistory.com/622?category=895441
https://jithub.tistory.com/360