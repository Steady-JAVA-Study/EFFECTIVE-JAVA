# ITEM 50. 적시에 방어적 복사본을 만들라

클라이언트가 불변식을 깨뜨리려한다고 가정하고 방어적으로 프로그래밍해야 한다.

악의적인 공격으로부터 방어하지 못하거나, 또는 실수로 클래스를 오작동하게 만들 수도 있다.

어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하다. 하지만 이는 주의를 기울여야 보장할 수 있다.

## 불변식을 지키지 못한 예제

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}
```
```java
public final class Period {
    // .. 코드 생략

    public static void main(String[] args) {
        Date start = new Date();
        Date end = new Date();
        Period period = new Period(start, end);
        end.setYear(78); // period의 내부를 수정했다!
    }
}
```

- 자바 8 이후로는 쉽게 해결할 수 있다. Date 대신 불변식인 Instant를 사용하거나 LocalDateTime이나 ZonedDateTime을 사용하면 된다.
- Date는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안 된다.
- 외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 한다. 그 다음 Period 인스턴스 안에서는 원본이 아닌 복사본을 사용한다.

방어적 복사에 Date의 clone 메서드를 사용하지 않은 것도 주목하자. Date 는 final 이 아니기 때문에 clone 이 Date 가 정의한 게 아닐 수도 있다. 즉, clone 이 악의를 가진 하위 클래스의 인스턴스를 반환할 수도 있다.

### 수정한 생성자 - 방어적 복사본

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }

    // 생략
}
```

## Period 인스턴스를 향한 접근자 메서드 공격

```java
public final class Period {

    // 코드 생략 ..

    public static void main(String[] args) {
        Date start = new Date();
        Date end = new Date();
        Period period = new Period(start, end);
        period.end().setYear(78); // period의 내부를 변경했다!
    }
}
```

### 수정한 접근자 - 필드의 방어적 복사본을 반환한다.

```java
public final class Period {

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
```

생성자와는 달리 접근자 메서드에서는 방어적 복사에 clone 을 사용해도 된다. Period가 가지고 있는 Date 객체는 java.util.Date임이 확실하기 때문이다(신뢰할 수 없는 하위 클래스가 아니라는 의미다.)

