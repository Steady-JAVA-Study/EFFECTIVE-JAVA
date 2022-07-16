# ITEM 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다. 가령 맞춤법 검사기는 사전(dictionary)에 의존하는데, 이런 클래스를 정적 유틸리티 클래스(아이템 4)로 구현한 모습을 드물지 않게 볼 수 있다.

## 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // 객체 생성 방지

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions (String typo) { ... }
}
```

## 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions (String typo) { ... }
}
```

위의 두 방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서 그리 훌륭해 보이지 않는다. 실전에서는 언어별로 있을 수도 있고, 특수 어휘용 사전을 별도로 두기도 한다.

SpellChecker가 여러 사전을 사용할 수 있도록 만들어보자. SpellChecker가 여러 사전을 사용할 수 있도록 만들기 위해 dictionary 필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가할 수 있으나 이 방법은 어색하며, 오류를 내기 쉽다. 또한 멀티스레드 환경에서는 사용할 수 없다.

## 의존 객체 주입 방식 - 유연성과 테스트 용이성을 높여준다.

```java
public class SpellChecker{
	private final Lexicon dictionary;
	public SpellChecker(Lexicon dictionary){
		this.dictionay = Objects.requiredNonNull(dictionay);
	}
	public boolean isValid(String word){...}
	public List<String> suggestions(String typo){...}
}
```

### 의존성 주입(Dependency Injection)

의존 객체 주입 방식을 활용한 디자인 패턴으로 의존성 주입(Dependency Injection)이 존재한다. 의존성 주입은 Spring 프레임워크의 3가지 핵심 프로그래밍 모델 중 하나이다. 외부에서 두 객체간의 관계를 결정해주는 디자인 패턴으로, 인터페이스를 사이에 두어 클래스 레벨에서 의존 관계가 고정되지 않도록 도와준다. 이러한 방식은 객체의 유연성을 늘려주고 객체간의 결합도를 낮출 수 있는 효과를 가지고 있다.