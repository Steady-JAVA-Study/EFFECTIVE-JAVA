# ITEM 17. 변경 가능성을 최소화 하라

> 불변 클래스
> 
> 클래스의 인스턴스 값을 수정할 수 없는 클래스
>
> 특징: 설계, 구현, 사용이 쉽다 / 오류에서 가변 클래스보다 훨씬 안전하다.

## 불변 클래스 구현 방법 

1. 객체의 상태를 변경하는 메서드를 제공하지 않는다. (`setter`)
2. 클래스를 확장할 수 없도록 한다. (final class)
3. 모든 필드를 final로 선언한다. -> Thread safe 
4. 모든 필드를 private으로 선언한다.
5. 자신 이외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

```java
public final class Money {
    private final String currency;
    private final int amount;
    private final String[] cards;

    public Money(String currency, int amount) {
        this.currency = currency;
        this.amount = amount;
    }

    public Money plus(int additionMoney) {
        return new Money(this.currency, this.amount + additionMoney);
    }

    public String[] getCards() {
        return cards.clone();
    }
}
```

1. 객체의 상태를 변경하는 메서드를 제공하지 않는다. (`setter`)
   1. 클래스의 필드 currency, amount 를 바꿀 수 있는 메서드가 없음.
   2. 돈을 늘리는 메서드는 새로운 Money 객체를 반환한다. -> 돈을 늘릴 때, 해당 객체의 값(속성)을 변경한다면 그 객체를 사용하는 다른 객체들이 영향을 받는다. 따라서 새로운 객체를 만들어 반환함으로써 side-effect 를 줄일 수 있다.

2. 클래스를 확장할 수 없도록 한다. (final class)
   1. 클래스를 final 로 지정하여 확장이 불가능하게 함.
3. 모든 필드를 final로 선언한다. 
4. 모든 필드를 private으로 선언한다.
5. 자신 이외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
   1. 가변 참조 객체 cards는 final로 지정하더라도 주소를 참조하므로 불변이 아니다.
   2. 가변 객체 cards를 얻어노느 필드는 clone()을 이용한다.
