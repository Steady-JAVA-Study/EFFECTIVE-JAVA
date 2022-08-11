# ITEM 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

대부분의 열거 타입 상수는 자연스럽게 인덱스(하나의 정수값)을 갖게 된다.

따라서 Java enum은 ordinal 이라는 메서드를 제공해 인덱스를 반환해준다.

그러나 ordinal은 사용하면 안된다. 그 이유를 살펴보도록 하겠습니다.

```java
pbulic enum Number {
	ONE, TWO, THREE, FOUR;

	pbulic int convertInt() {
		return ordinal() + 1;
	}
}
```

위와 같은 Number라는 Enum에서 int형을 ordinal로 정의한다면 ONE -> 1, Two -> 2, ...의 인덱스를 갖게 된다.

만약 ZERO가 필요하다면 아래와 같은 코드가 될 것 이다.

```java
pbulic enum Number {
	ZERO, ONE, TWO, THREE, FOUR

	pbulic int convertInt() {
		return ordinal() + 1;
	}
}
```

ZERO -> 1, ONE -> 2, ...를 반환하게 된다.

## 해결방법

위와 같은 문제를 해결하기 위해서는 ordinal 메서드를 사용하지 않고 필드로 선언해준다.

```java
pbulic enum Number {
	ONE(1), TWO(2), THREE(3), FOUR(4);
	
	private final int number;

	public Number(int number) {
		this.number = number;
	}

	public int getNumber() {
		return number;
	}
}
```