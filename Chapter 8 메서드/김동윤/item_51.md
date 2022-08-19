# 메서드 시그니처를 신중히 설계하라

# 메서드 시그니처 설계
## 1) 메서드 이름을 신중히 
- 항상 표준 명명 규칙을 따라야 함
- 이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는게 최우선 목표
- 긴 이름은 피하고, 애매하면 자바 라이브러리의 API 가이드를 참조

## 2) 편의 메서드를 너무 많이 만들지 X
- 모든 메서드는 각자 자신의 소임을 다해야한다. 메서드가 너무 많은 클래스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 어렵다. 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공
## 3) 매개변수 목록은 짧게 유지 
- 매개변수가 4개 이상 넘어가면 이를 기억하기가 쉽지 않다. 같은 타입의 애개변수 여러개가 연달아 나오는 경우는 더 해롭다. 사용자가 매개변수 순서를 기억하기 어려울뿐더러, 실수로 순서를 바꿔 입력해도 그대로 컴파일되고 실행된다. 단지 의도와 다르게 동작

# 긴 매개변수 목록을 짧게
## 1) 여러 메서드로 쪼갠다, 쪼개진 메서드 각각은 원래 매개변수 목록의 부분 집합을 받는다.
```java
List<String> list = Lists.of("a", "b", "c", "d");

List<String> newList = list.subList(1, 3);
int index = newList.indexOf("b"); // 0

```
- (EX) 리스트에서 지정된 범위의 부분 리스트에서 특정 요소를 찾는다고 가정
- 이 기능을 하나의 메서드로 구현하려면 리스트의 시작과 끝 그리고 찾을 요소까지 총 3개의 매개변수가 필요
- 하지만 두 개의 메서드 (부분 리스트, 특정 인덱스 찾기)로 나누면 위와 같이 매개변수를 각 메서드에 2개 1개로 나누기 가능 

## 2) 매개변수를 묶어주는 도우미 클래스를 만든다.

```java
// 기존 메서드
public void someMethod(String name, String address, String email, String job) {
    // do something
}

// Helper 클래스 적용
class SomeHelper {
    String name;
    String address;
    String email;
    String job;
}

public void someMethod(SomeHelper someHelper) {
    // do something
    }
    
```

- 여러 매개변수 몇개를 독립된 하나의 개념으로 볼 수 있을때 추천하는 기법
- (EX) 카드 게임을 클래스로 만든다고 했을때 카드의 숫자(rank), 무늬(suit)를 뜻하는 두 매개변수를 항상 같은 순서로 전달
- DTO를 사용해서 매개변수를 넘기는 방식

## 3) 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용 ( 1,2 합친 것 )
- 모든 매개변수를 하나로 추상화한 객체로 정의
- 클라이언트에서 이 객체의 세터(setter) 메서드를 호출하여 필요한 값을 설정

```
1) 모든 매개변수를 하나로 추상화한 객체를 만든다. (2번 테크닉에서 다룬 카드, 좌표 등)
2) setter 메서드로 필요한 매개변수를 세팅
3) execute 메서드로 설정한 매개변수의 유효성 검사
4) 해당 객체를 넘겨 원하는 계산 수행

```

# 매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다.
- 매개변수로 적합한 인터페이스가 존재한다면 인터페이스를 직접 사용 
- (EX) 메서드에 HashMap을 넘기지 말고  Map을 넘겨라. 그러면 HashMap 뿐만 아니라 다른 Map 구현체도 인수로 건넬 수 있다.
- 인터페이스 대신 클래스를 사용한다면 클라이언트에게 특정 구현체만 사용하도록 제한, 따라서 인터페이스 사용이 better 

# boolean보다는 원소 2개짜리 열거 타입이 낫다.


```java
public enum TemperatureScale {FARENHEIT, CELSIUS} 
Thermometer.newInstance(true); 
Thermometer.newInstance(TemperatureScale.CELSIUS);
```

- 열거 타입을 사용한다면 코드를 읽고 쓰기가 쉬워 
- 선택지를 추가하는것도 간편 

- 온도계 클래스가 있다고 가정
- TemperatureScale(true) 보다 TemperatureScale.newInstance(TemperatureScale.CELSIUS)가 하는 일을 명확히 알려줌


## 결론 
- 메서드 이름을 신중히 
- 편의 메서드를 너무 많이 만들지 X
- 매개변수 목록은 짧게 유지해야 한다.
- 매개변수의 타입으로는 클래스보다는 인터페이스가 낫다.
- boolean보다는 원소 2개짜리 열거 타입이 낫다.


## reference
https://gona.tistory.com/73
https://devfunny.tistory.com/621?category=895441
https://daebalpri.me/entry/Effective-Java-3E-ITEM-51-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%8B%9C%EA%B7%B8%EB%8B%88%EC%B3%90%EB%A5%BC-%EC%8B%A0%EC%A4%91%ED%9E%88-%EC%84%A4%EA%B3%84%ED%95%98%EB%9D%BC
