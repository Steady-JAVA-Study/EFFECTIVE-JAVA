# ITEM 34. int 상수 대신 열거 타입을 사용하라

열거 타입(enumerated type)은 고정된 상수의 집합으로 구성된다.

열거 타입이 자바에 추가되기 전에 가장 흔하게 사용하던 패턴은 **열거 패턴(enum pattern)** 이다.

열거 패턴의 구현 방법은 `static final int`로 값을 정의하는 것이다.

```java
public static final int name = 0;
public static final int departmentName = 1;
public static final int companyName = 2;
public static final int phoneNumber = 3;
public static final int email 4;
```

아래와 같은 단점이 존재한다.

1. 타입 안전을 보장할 수 없다.
2. 표현력도 좋지 않다.
3. 동등 연산자(==)로 비교하더라도 컴파일러가 아무런 경고 메시지를 출력하지 않는다.
4. 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨지기 때문에, 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다.

## 열거형 타입(EnumType)

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD}
```

열거 타입 또한 클래스로 취급되어, 각 필드마다 하나의 인스턴스로 구성된다.

각 필드는 public static final로 정의된다.

싱글톤으로 구현되어 있다.

열거 타입은 컴파일 타임에 안전하다는 장점을 가진다. `Apple` 이라는 열거 타입이 기대되는 자리에 다른 열거 타입은 올 수 없으며, null도 올 수 없다.

> Enum을 선언하는 경우
>
> 타입으로 분류될 수 있는 2개 이상의 값이 있다.
>
> 당장은 한계가 있는데 확장될 가능성이 존재한다.


```java
@AllArgsConstructor
@Getter
public enum Fruit {
    APPLE(1000, 20, "RED"),
    PEACH(200, 9, "YELLO");

	private final int price;
	private final int box;
	private final String color; 
    
    public int boxPrice() {
        return price * box;
    }
}
```

### values

EnumType은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다.

```java
Fruit.values() // APPLE, PEACH
```

### 상수별 메서드 구현(constant-specific method implementaltion)

```java
public enum Operation {
    PLUS("+") {
	    public douvle apply(double x, double y) { return x + y };
	},
	MINUS("-") {
		public douvle apply(double x, double y) { return x - y };
	},
	TIMES("*") {
		public douvle apply(double x, double y) { return x * y };
	},
	DIVIDE("/") {
		public douvle apply(double x, double y) { return x / y };
	};
	
	private final String symbol;
	Operation(String symbol) {this.symbol = symbol;}
	@Override public String toString() {return symbol;}
	public abstract double apply(double x, double y);
}
```

EnumType은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다.

EnumType에 apply라는 추상 메서드를 선언하고 각 상수에서 자신에 맞게 재정의하는 방법이다.

---

## fromString method(자주 쓰이는 pattern)

String으로 response된 값들을 Enum으로 변환해야하는 경우가 종종 발생한다.

```java
private static final Map<String, Fruit> stringToEnum =
    Stream.of(values())
        .collect(Collectors.toMap(Objects::toString, e->e));
public static Optional<Fruit> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol))
        }
```