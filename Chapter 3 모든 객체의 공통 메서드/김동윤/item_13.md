.# clone 재정의는 항상 주의해라
## Cloneable 인터페이스!?

> - 복제해도 되는 클래스임을 명시하는 Mixin 인터페이스
- 믹스인 : 다른 클래스의 부모클래스가 되지 않으면서 다른 클래스에서 사용할 수 있는 메서드를 포함하는 클래스 ('상속' 이 아닌 '포함')
- 객체 복사를 하기 위해서는 반드시 clone 메서드를 이용

```java
public interface Cloneable {}
```

### Cloneable 쓰는 이유 & 이는 clone 을 구현 needed 
- clone() 은 모든 클래스에서 오버라이딩 할 수 있다.
- 하지만 Cloneable 인터페이스를 구현하지 않은 클래스의 clone() 은 예외를 던짐
- super.clone() 을 사용하지 않을 거라면, Cloneable 필요 없음.
- Cloneable 을 구현한 클래스만 clone() 을 사용할 수 있다.
- clone() 은 오버라이딩하는 메소드이다.
- Cloneable 은 상위 클래스 (Object) 에 정의된 clone() 의 동작방식을 변경한다는 의미이다.

### Cloneable , clone 규약~ 
- x.clone() != x 는 참이여야 한다.

- x.clone().getClass() == x.getClass() 는 참이여야 한다.

- x.clone().equals(x) 는 참이여야 한다.

- 기초자료형을 필드로 담고 있을 경우 쉽게 지켜지기 possible

=> 위 규약은 선택 옵션임. 
_____________
### 생성자 연쇄
- 어떤 클래스에서 clone() 을 new 키워드를 통해 복사하는 방식으로 재정의했다고 생각해보자 & 이러한 재정의는 컴파일상에 문제 X
- BUT  그 클래스를 상속한 클래스에서 super.clone() => 하위 클래스의 타입이 아닌 상위 클래스의 타입 객체를 반환
- 클래스가 final 이면 하위 클래스가 없으니 new 를 써도 괜찮다. (하지만 Cloneable 구현이유도 없다)
- 참조형은 CLONE 사용하기 어려움 
### 추천 방법
복제용 생성자나 복제용 정적 팩토리 메서드를 만드는 것을 권장 

## 출처
이펙티브 자바 도서 
https://github.com/Meet-Coder-Study/book-effective-java