# ITEM 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

## 상수 인터페이스(Constant static final)은 안티 패턴!

```java
public interface BlackjackConstants {
    
    static final int BLACK_JACK_NUMBER = 21;
    
    static final int ALTERNATE_ACE_VALUE = 11;
    
    static final int DEALER_MIN_VALUE = 17;
}
```

상수 인터페이스는 사용하지 않을 수도 있는 상수를 포함하여 모두 가져오기 때문에 계속 가지고 있어야 한다.

클라이언트가 상수 인터페이스 값에 직접 접근할 수 있다. 이때, 불필요한 정보를 노출하는 것이 오히려 클라이언트에게 혼란을 야기할 수 있다.

차라리 클래스에 static final로 추가하는 것이 더 낫다.

```java
public class OrderService {
    public static final double SECOND_TO_MIN = 60;
}
```

