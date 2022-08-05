# int 상수 대신 열거 타입을 사용하라

열거 타입 : 고정된 상수의 집합, 그 외의 값은 허용하지 않는 타입

### 열거 타입 등장 전
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
< 단점 >
- 타입 안전 보장 x
- 표현력 좋지 않음
   - 컴파일러가 이해하는 값은 정수이므로 오렌지를 건네야할 메서드에 사과를 보내고 동등연산자로 비교하더라도 컴파일러는 틀린것을 모름
- 상수의 값이 바뀌면 클라이언트도 다시 컴파일해야함
- 문자열로 출력하기가 까다로움

- 깃허브 이펙티브 자바에서의 좋은 관련 예시 코드([코드 출처](https://github.com/Meet-Coder-Study/book-effective-java/blob/main/6%EC%9E%A5/34_int_%EC%83%81%EC%88%98_%EB%8C%80%EC%8B%A0_%EC%97%B4%EA%B1%B0_%ED%83%80%EC%9E%85%EC%9D%84_%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%ED%99%A9%EC%A4%80%ED%98%B8.md))

```java
//정수 열거 패턴(따라하지 말것!)
public class Constants {
    public static final int MONDAY = 0;
    public static final int TUESDAY = 1;
    public static final int WEDNESDAY = 2;
    public static final int THURSDAY = 3;
    public static final int FRIDAY = 4;
    public static final int SATURDAY = 5;
    public static final int SUNDAY = 6;

    public static final int APPLE = 0;
    public static final int ORANGE = 1;
    public static final int GRAPE = 2;
    public static final int MELON = 3;
}
--------------------------------------------------------------
//열거 타입
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
public enum Fruit {
    APPLE, ORANGE, GRAPE, MELON
}
```

### 정수열거 패턴일 경우 
```java
public void function(int day) { //요일 관련 상수만 들어가야 하는 메서드
    ...
}

...
function(MONDAY); //OK
function(APPLE); //과일 관련 상수가 들어가도 컴파일러는 모른다!
```

### 열거 타입 짱짱맨
```java
public void function(Day day) { //요일만 들어가야 하는 메서드
    ...
}

...
function(MONDAY); //OK
function(APPLE); //컴파일 에러!
```

### 문자열 출력

### 1) 정수 열거 패턴
```java
System.out.println(Constants.MONDAY); //0
```

### 2) 열거 타입
```java
System.out.println(Day.MONDAY); //MONDAY
```

## 열거 타입(enum type) 의 등장
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD}
```

생긴건 다른 언어의 열거 타입과 비슷해보이지만, 자바의 열거 타입은 다르다.

### < 열거 타입 > 
- 열거 타입은 클래스
- 상수 하나당 자신의 인스턴스를 하나씩 만들어(Singleton) public static final 필드로 공개 
- 열거 타입은 final => 밖에서 접근가능한 생성자를 제공 X
- 열거 타입 선언으로 만들어진 인스턴스는 딱 1개만 존재 

### < 열거 타입과 싱글턴 >
- 열거 타입은 인스턴스 통제클래스,  언제 어느 인스턴스를 살아 있게 할지를 통제 가능 (정적 팩터리 방식 클래스)
- 싱글턴 = 원소가 하나뿐인 열거타입
- 열거타입 = 싱글턴을 일반화한 형태

### < 열거타입의 장점 >
- 컴파일 타입 안전성 제공 : Apple 열거타입 인수에 Orange를 넘길 수 없음
- 열거 타입에 새로운 상수를 추가하거나, 순서를 바꿔도 다시 컴파일 하지 않아도 OK 
- 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현 가능


## 데이터와 메서드를 갖는 열거 타입
- 실제로는 클래스여서 고차원의 추상 개념 하나를 표현 가능
- 각 상수와 연관된 데이터를 해당 상수 자체에 내재
- 상수 별로 데이터를 가질 수 있고, enum에서 공통적으로 사용하는 메서드를 선언하거나 상수 별로 메서드를 재정의 가능 

### 1. 열거 타입 상수 각각을 특정 데이터와 연결
```java
@Getter
public enum Planet {
    MERCURY(3.302e+23, 2.439e6), //mass, radius 
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.447e7);
    
    private final double mass;            // 질량(단위: 킬로그램)
    private final double radius;          // 반지름(단위: 미터)
    private final double surfaceGravity;  // 표면중력(단위: m / s^2)
    
    // 중력상수 (단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;
    
    // 생성자
    // 열거 타입 상수 각각을 특정 데이터와 연결지으려면 
    // 생성자에서 데이터를 받아 인스턴스 필드에 저장 
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }
    
  
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}

```

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 **생성자에서 데이터를 받아 인스턴스 필드에 저장** 
- 열거 타입은 외부로의 생성자를 제공하지 않기 때문에 불변 객체 
- 모든 필드는 final 
- 필드를 public으로 선언해도 되지만, private로 선언 후 접근자 메서드를 제공하는 것 gooood

### 나의 enum 코드
```java
package eci.server.ItemModule.entity.item;


public enum ItemType {

    NONE(0),
    자가개발(0), //자가개발
    원재료(1), //원재료
    단순외주구매품(2), // 외주구매품(단순)
    시방외주구매품(3), //외주구매품(시방)
    사내가공품(3), // (외주구매품 시방이랑 THE SAME)
    프로덕트제품(4),//제품
    파트제품(4),//제품
    기타(0), //기타 - 엮일 것 없삼
    부자재(1) //부자재
    ;

    private final Integer label;

    ItemType(Integer label){
        this.label = label;
    }

    public Integer label() {
        return label;
    }
}

```

#### (+) Enum 멤버 추가하기
- Enum에 멤버를 추가하는 이유는 enum 상수와 연관된 데이터를 상수 자체에 포함시켜 용이하게 관리하기 위함
- 인스턴스 필드를 추기하려면 enum 상수의 이름 옆에 원하는 값을 괄호()와 함께 적어주면 ok
```java
public enum SearchSite {
	NAVER("https://www.naver.com"),
	DAUM("https://www.daum.net"),
	GOOGLE("https://www.google.com"),
	BING("https://www.bing.com");
 
	private final String url; // 인스턴스 필드 추가
 
	// 생성자 추가
	SearchSite(String url) { this.url = url; }
 
	// 인스턴스 필드 get메서드 추가
	public String getUrl() { return url; }
}
```

## 상수별 메서드 구현
```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE
}
```
- 위와 같은 사칙 연산 열거타입이 있을 때, 사칙연산에 대한 로직을 열거 타입 내에 작성하고 싶을 때가 있지~

### 1) switch-case을 이용한 구현
```java
public double apply(double x, double y) {
    switch(this) {
        case PLUS: return x + y;
        case MINUS: return x - y;
        case TIMES: return x * y;
        case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
}
```

### 2) 상수 한정 메서드 구현
```java
public enum Operation {
  PLUS {public double apply(double x, double y) { return x + y; }},
  MINUS {public double apply(double x, double y) { return x - y; }},
  TIMES {public double apply(double x, double y) { return x * y; }},
  DIVIDE {public double apply(double x, double y) { return x / y; }};
  public abstract double apply(double x, double y);
}

```
- 위와 같이 선언하면 상수가 추가 될 때마다 apply 메서드를 재정의
- 위의 식을 람다식으로 정리하면
```java
public enum Operation {
    PLUS((x, y) -> x + y),
    MINUS((x, y) -> x - y),
    TIMES((x, y) -> x * y),
    DIVIDE((x, y) -> x / y);
    
    private BiFunction<Double, Double, Double> operationFuntion;   
}
```

## 전략 열거 타입 패턴
- 열거 타입 상수 일부가 같은 동작을 공유할 때 사용
(ex)  요일별로 일당을 계산해 주는 열거타입 메서드

### switch-case 문 
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;
        switch(this) {
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = 
                minutesWorked <= MINS_PER_SHIFT ? 0 : 
                	(minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```
- 간결하지만, 관리 관점에서는 위험한 코드 
- 공휴일과 같은 새로운 값을 열거타입에 추가하려면 그 값을 처리하는 case문을 잊지말고 추가 필요

> 대안 : 평일용/주말용으로 나눠 각각 도우미 메서드를 생성한 다음, 각 상수가 자신에게 필요한 메서드를 적절히 호출하게 해보자 

### 
```java
enum PayrollDay {
    MONDAY(WEEKDAY), //payType 이라는 전략을 선언하게 했음 
    TUESDAY(WEEKDAY),
    WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY),
    FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND),
    SUNDAY(WEEKEND);
    
    // 인스턴스 필드 추가
    private final PayType payType;
    
    // 생성자 추가
    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    enum PayType { 
        WEEKDAY { //각 enum이 선택한 전략에 따라서 메소드 달라지지
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ? 0 :
                (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2
            }
        };
        
        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked & payRate;
            return basePay + overtimePay(minutesWorked, payRate);
        }
    }
}
```

## 열거 타입을 언제 사용할까?
- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
(ex) 태양계 행성, 한 주의 요일, 체스말


## 결론
- 열거 타입은 정수 상수보다 안전하고 가독성이 좋고 강력 
- 열거 타입의 각 상수를 특정 데이터와 연결짓거나 상수별로 동작을 다르게 할 때 명시적 생성자나 메서드가 쓰인다.
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요 X
- 하나의 메서드가 상수별로 다르게 동작해야할 때는 switch 문 대신에 상수별 메서드 구현을 사용 (추상메서드 재정의)
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용 



### reference 
https://jyami.tistory.com/102
https://jaehun2841.github.io/2019/02/03/effective-java-item34/#abstract-method%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B5%AC%ED%98%84