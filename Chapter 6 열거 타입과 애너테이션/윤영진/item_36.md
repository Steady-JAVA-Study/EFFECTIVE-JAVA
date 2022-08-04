# ITEM 36. 비트 필드 대신 EnumSet을 사용하라

## 비트 필드

```java
public class Text {
    public static final int BOLD = 1 << 0; // 00000001 -> 1
    public static final int ITALIC = 1 << 1; // 00000010 -> 2
    public static final int UNDERLINE = 1 << 2; // 00000100 -> 4
    public static final int STRIKETHROUGH = 1 << 3; // 00001000 -> 8

    public void applyStyles(int styles) {
        // ...
    }
}
```

여러 상태값을 조합해서 하나로 관리해야하는 경우 -> BOLD AND ITALIC

> BOLD | ITALIC을 2, ITALIC을 3 BOLD | UNDERLINE을 4, ... 이런식으로 관리하면 되는거 아니야? 
> 
> 유지보수에 너무 큰 단점을 지님, 따라서 조합을 이용하자


클래스 안에 상수로 표현할 때 쉬프트 연산을 사용해 비트로 표현하는 것이다. 이렇게 사용하면 상수 값들을 비트로 표현할 수 있다.

```java
text.applyStyles(BOLD | ITALIC); // 0001 | 0010 -> 3
```

위와 같이 비트의 OR연산을 통해 여로 상수를 하나의 집합으로 모을 수 있다. 이렇게 만들어진 집합을 비트 필드라고 한다.

위 예시에서도 BOLD와 UNDERLINE이 적용된 값이 3인데 3만 보고서는 이를 파악하기 매우 힘들다. 비트 필드 하나에 녹아있는 모든 원소 순회도 까다롭다. 마지막으로 필요한 최대 비트를 API 작성시 예측하여 int나 long 같은 적절한 타입을 선택해야한다.

위와 같은 단점을 보안하기 위해 `EnumSet` 이 사용된다.

## EnumSet

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```

> 원소가 64개 이하라면 RegularEnumSet, 65개 이상이면 JumboEnumSet을 사용

Enum과 EnumSet을 활용해서 위 정수 열거 패턴의 Text를 다음과 같이 변경할 수 있다.

```java
@Getter
@AllArgsConstructor
public class Text {
    public enum Style {
        BOLD(1), ITALIC(2), UNDERLINE(4), STRIKETHOUGH(8)
    }
    
    private final int code;
    
    public static int getStyleCode(Set<Style> styles) {
        return styles.stream().mapToInt(Style::getCode).sum();
    }

}

int styleCode = Style.getStyleCode(EnumSet.of(Style.BOLD, Style.ITALIC));
```