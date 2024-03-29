# 필요 없는 검사 예외 사용은 피하라

## 검사 예외의 장점
- 검사 예외를 싫어하는 자바 프로그래머가 많지만 제대로 활용하면 API와 프로그램의 질을 높이기 가능함
- 결과를 코드로 반환하거나 비검사 예외를 던지는 것과 달리, 검사 예외는 발생한 문제를 프로그래머가 처리하여 안전성을 높이게끔 해줌

## 검사 예외의 문제
- 물론, 검사 예외를 과하게 사용하면 오히려 쓰기 불편한 API 
- 어떤 메서드가 검사 예외를 던질 수 있다고 선언됐다면, 이를 호출하는 코드에서는 catch 블록을 두어 그 예외를 붙잡아 처리하거나 더 바깥으로 던져 문제를 전파해야 함
- 검사 예외를 던지는 메서드는 스트림 안에서 직접 사용할 수 없기 때문에 자바 8부터는 부담

## 비검사 예외
- API를 제대로 사용해도 발생할 수 있는 예외이거나, 프로그래머가 의미 있는 조치를 취할 수 있는 경우라면 이 정도 부담쯤은 받아들일 수 있을 것 
- 그러나 둘 중 어디에도 해당하지 않는다면 비검사 예외를 사용하는 게 좋다. 
- 검사 예외와 비검사 예외 중 어느 것을 선택해야 할지는 프로그래머가 그 예외를 어떻게 다룰지 생각해보면 알 수 있음

- 다음과 같이 하는 것이 최선 ?

```java
} catch (TheCheckedException e) {    
throw new AssertionError(); 
// 일어날 수 없다!
}
```

- 아니면 다음 방식은  ?

```java
} catch (TheCheckedException e){    
	e.printStackTrace(); 
	System.exit(1);
    }
```

=> 더 나은 방법이 없다면 비검사 예외를 선택


## 검사 예외를 회피하는 방법
- 검사 예외가 프로그래머에게 지우는 부담은 메서드가 단 하나의 검사 예외만 던질 때가 특히 큼
- 이미 다른 검사 예외도 던지는 상황에서 또 다른 검사 예외를 추가하는 경우라면 기껏해야 catch 문 하나 추가하는 선에서 끝 
- 하지만 검사 예외가 단 하나뿐이라면 오직 그 예외 때문에 API 사용자는 try 블록을 추가해야 하고 스트림에서 직접 사용하지 못하게 된다. 
=> 그러니 이런 상황이라면 검사 예외를 안 던지는 방법이 없는지 고민해볼 가치 있음 

### 적절한 결과 타입을 담은 옵셔널을 반환하는 방법
- 검사 예외를 던지는 대신 단순히 빈 옵셔널을 반환
- 예외가 발생한 이유를 담을 수 없다는 단점 
   - 예외를 사용한다면 구체적인 예외 타입과, 부가 정보를 제공할 수 있다.

### 검사 예외를 던지는 메서드를 두개로 쪼개 비검사 예외로 변환 
- 상태 검사 메서드와, 상태 의존 메서드로 나누는 방법 
- 유연한 방법이지만 상태 검사 메서드의 단점이 그대로 적용 

이 방식에서 첫 번째 메서드는 예외가 던져질지 여부를 boolean 값으로 반환

- 검사 예외를 던지는 메서드 - 리팩터링 전

```java
try {    
	obj.action(args);
} 
catch (TheCheckedException e) { 
... // 예외 상황에 대처한다.
}
```

리팩터링하면 다음처럼 된다.
상태 검사 메서드와 비검사 예외를 던지는 메서드 - 리팩터링 후

```java
if (obj.actionPermitted(args)) {   
	obj.action(args);
} 
else {    
	... // 예외 상황에 대처한다.
    }
```

- 이 리팩터링을 모든 상황에 적용할 수는 없다. 그래도 적용할 수만 있다면 더 쓰기 편한 API를 제공할 수 있다. 
- 리팩터링 후의 API가 딱히 더 아름답진 않지만, 더 유연한 것은 확실하다. 
- 프로그래머가 이 메서드를 성공하리란 걸 안다거나, 실패 시 스레드를 중단하길 원한다면 다음처럼 한 줄로 작성해도 무방하다.
```
obj.action(args);
```
- 이 한줄 짜리 호출 방식이 주로 쓰일 거로 판단되면 리팩터링하는 편이 바람직하다. 

- 한편, actionPermitted는 상태 검사 메서드에 해당하므로 아이템 69에서 말한 단점도 그대로 적용되니 주의해야 한다. 즉, 외부 동기화 없이 여러 쓰레드가 **동시에 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있다면 이 리팩터링은 적절하지 않다.** actionPermitted와 action 호출 사이의 객체의 상태가 변할 수 있기 때문이다.
- 또한 actionPermitted가 action 메서드의 작업 일부를 중복 수행한다면 성능에서 손해이니, 역시 이 리팩터링이 적절하지 않을 수 있다.


### reference 
https://hirlawldo.tistory.com/127