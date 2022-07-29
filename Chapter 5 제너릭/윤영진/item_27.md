# ITEM 27. 비검사 경고를 제거하라

컴파일러가 보내는 warning을 제거하라

아래 코드와 같이 구현체의 타입을 명확히 주지 않은 채로 HashSet 객체를 생성할 수 있다.

`Set<String> set = new HashSet()`;

위 코드에서는 다음과 같은 경고를 볼 수 있다.

```text
Raw use of parameterized class 'HashSet'
Unchecked assignment: 'java.util.HashSet' to 'java.util.Set<java.lang.String>'
```

즉, 인스턴스로 생성한 `HashSet`을 Raw 타입으로 사용했기 때문에 나타난 비검사 경고이다. 이를 해결하기 위해서는 HashSet<String>과 같이 타입 매개변수를 명시하던가 자바 7부터 지원하는 제네릭 타입 추론을 활용하여 다이아몬드 연산자 <>를 사용할 수 있다.

위와 같은 비검사 경고를 가능한 모두 제거해야한다. 비검사 경고를 없애면 타입 안정성이 높아지고 ClassCastException이 발생할 여지를 없애준다.

## 비검사(unchecked)의 의미

비검사(unchecked)는 Java 컴파일러가 type-safe를 보장하기 위해 필요한 타입 정보가 충분히 존재하지 않는다는 의미이다. 기본적으로 Java 컴파일러는 비검사 경고를 활성화하지 않으며 이를 사용하기 위해서는 컴파일러에 -Xlint:uncheck 옵션을 주어야한다.

## 비검사 경고를 제거할 수 없다면?

모두들 최대한 경고를 없애도록 노력할 것 이다. 경고를 제거할 수 없지만 타입이 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchcked")` 를 달아 경고를 숨길 수 있다.

