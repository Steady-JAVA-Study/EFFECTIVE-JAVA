# ITEM 51. 메서드 시그니처를 신중히 설계하라

> 메서드 시그니처?
> 
>  메서드의 이름과 매개변수의 순서, 타입, 개수를 의미한다. 메서드의 리턴 타입과 예외 처리하는 부분은 메서드 시그니처가 아니다.

## 메서드 이름은 신중히 짓자.

항상 표준 명명 규칙(Item 68)을 따라야 한다. 이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는 게 최우선 목표다.

긴 이름은 피하도록 한다. 애매하면 자바 라이브러리의 API 가이드를 참조하도록 한다. 대부분은 납득할만한 수준이다.

## 편의 메서드(Util Class 소속의 method 와 같은)를 너무 많이 만들지 말자.

메서드가 너무 많은 클래스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 어렵다. (인터페이스도 마찬가지)

정말 자주 쓰일 때에만 편의 method를 제공해야 한다.

확신이 생기지 않는다면 private로 쪼개 놓은 후 차후에 다른 class에서의 니즈가 생길 경우 리팩토링해도 늦지 않다.

## 매개변수 목록은 짧게 유지하자.

**4개 이하를 유지하자**

- 4개가 넘어가면 매개변수를 전부 기억하기 어렵다.
- 같은 타입의 매개변수 여러 개가 연달아 나오는 경우는 특히 해롭다.

### 매개변수를 줄여주는 3가지 기술

1. 여러 메서드로 쪼기기

2. 매개변수 여러 개를 묶어주는 도우미 클래스를 만들기
   - 일반적으로 이런 도우미 클래스는 정적 멤버 클래스로 둔다.

3. 1번과 2번의 혼합형태, 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다

```java
public void inputGamer(String name, double height, double weight, boolean hair, int age){
	/*
		Logic
		매개변수가 너무 많다.
	*/
}

// 호출시 //
// 뭐가 먼저 나와야하는지 헷갈림
```
```java
public void inputGamer(Person person){
		// Logic
}

// 호출시 //
// 원하는 값만 순서에 상관없이 설정 가능
cardGame.inputGamer(Person.builder()
      .height(199.99)
      .weight(99.02)
      .name("Foo bar").build());

cardGame.inputGamer(Person.builder()
	    .name("Foo bar")
	    .height(199.99)
	    .weight(99.02)
	    .hair(false)
	    .build());

// 유효성 검사도 가능
cardGame.inputGamer(Person.builder()
	    .height(199.99)
	    .weight(-99.02)
	    .build()); // ERROR

// Person class //
@Builder(builderClassName = "Builder")
public class Person {
    private String name;
    private double height;
    private double weight;
    private boolean hair;
    private int age;

    public static class Builder {
        Person build() {
            if (height < 0 || weight < 0 || age < 0){
                throw new IllegalArgumentException("Invalid data: data is greater than 0");
            }
            return new Person(name, height, weight, hair, age);
        }
    }
}

```

## 매개변수의 타입으로는 클래스보다는 인터페이스를 사용하라.

인터페이스 대신 클래스를 사용하면 클라이언트에게 특정 구현체만 사용하도록 제한하게 된다

예를 들어 HashMap 대신 Map을 사용, ArrayList 대신 List를 사용하는 것이다.

##  boolean보다는 원소 2개짜리 열거 타입이 낫다

열거 타입(enum)을 사용하면 코드를 읽고, 쓰기가 더 쉬워진다.

```java
enum TemperatureScale {
  FAHRENHEIT,
  CELSIUS
}
```

FAHRENHEIT: true, CELSIUS: false 로 지정하는것 보다 enum이 훨씬 명확하다.

또한 나중에 다른 온도 단위를 추가한다면 리팩터링하기 편하다.

