# 비트 필드 대신 EnumSet을 사용하라

## 비트 필드(bit field)
- 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모을 수 있으며,
이렇게 만들어진 집합 
- Enum이 없던 시절에도 상수들을 집합(Set)처럼 관리해야될 때 사용됐음
```java
public class Text {
    public static final int BOLD = 1 << 0; // 첫 번째 비트 // 1 // 이진수 : 0001
    public static final int ITALIC = 1 << 1; // 두 번째 비트 // 2 // 이진수 : 0010
    public static final int UNDERLINE = 1 << 2; // 세 번째 비트 // 4 // 이진수 : 0100
    public static final int STRIKETHROUGH = 1 << 3; // 네 번째 비트 // 8 이진수 : 1000

    public void applyStyles(int styles) {
        // ...
    }
}
```
- 클래스 안에 상수로 표현할 때 쉬프트 연산을 사용해 비트로 표현하는 것 
- 이렇게 사용하면 상수 값들을 비트로 표현 가능
- 비트값으로 표현된 Enum들에게 OR 비트 연산자(|)를 넣어서 Enum 값들을 합쳐서 표현할 수 있음 
```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC); // 1 | 2 => 3
```

< 단점 > 

- 컴파일 되면 값이 새겨짐 (무슨 의미인지 모름)
   - applyStyles() 안에서 3이란 값으로 로직을 실행해야 하는데 3이라는 숫자는 BOLD, ITALIC 이라는 정보를 가지고 있지 X
- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수보다 해석 어려움
(어떤 값이 OR연산되서 나온 값인지 알기 어렵지)
- 최대 몇 비트가 필요한지 API작성 시 미리 예측이 필요 

## 비트 필드 대신 EnumSet을 사용
```java
public enum Text {
    BOLD, ITALIC, UNDERLINE, STRIKETHROUGH ...
}
```

- java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현 
- Set 인터페이스를 완벽히 구현 , 타입 안전하고 다른 Set 구현체와도 함께 사용 
- EnumSet의 내부는 비트 벡터(- EnumSet의 내부는 비트 벡터(arithmetic bitwise operations)로 구현 (These computations are very fast and therefore all the basic operations are executed in a constant time & use less memory)
- 원소가 64개 이하인 경우에는 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여주지 
- removeAll과 retailAll과 같은 대량 작업은 비트를 효율적으로 처리 할 수 있는 산술 연산을 사용 

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 어떤 Set을 넘겨도 되나 EnumSet이 가장 좋다.
    // 이왕이면 Set Interface로 받는게 좋은 습관이다.
    public void applyStyles(Set<Style> styles) {...}
}
```

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

## enumset 가지구 set 이 제공하는 메소드 다 사용가능하지

```
EnumSet<Color> set = EnumSet.noneOf(Color.class);
set.add(Color.RED);
set.add(Color.YELLOW)
Check if the collection contains a specific element:

set.contains(Color.RED);
Iterate over the elements:

set.forEach(System.out::println);
Or simply remove elements:

set.remove(Color.RED);
```

## 결론 
- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도, 비트 필드를 사용 이유는 없음
- EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공  
### reference
https://www.baeldung.com/java-enumset