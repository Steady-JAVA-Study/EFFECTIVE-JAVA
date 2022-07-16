.# Comparable 구현을 고려하라

## 인트로 : Comparator vs Comparable
- 내가 기억이 안나서 작성하는~~ 인트로 부분
### Comparable : 정렬 수행 시 기본적으로 적용되는 정렬 기준이 되는 메서드

> - 인터페이스를 구현한 객체 스스로에게 부여하는 한 가지 기본 정렬 규칙을 설정

### Comparator : 정렬 가능한 클래스(Comparable 인터페이스를 구현한 클래스)들의 기본 정렬 기준과 다르게 정렬 하고 싶을 때 사용하는 인터페이스

> - 인터페이스를 구현한 클래스는 정렬 규칙 그 자체

# Comparable 구현할 때 
- 기본 정렬 규칙과는 다르게 원하는대로 정렬순서를 지정할 때 사용

- 자연적인 순서를 고려해야 한다면 Comparable 구현을 추가하라
(자연적인 순서 : 자연적인 순서가 있다는 것은 정렬 기능을 추가로 사용 가능 / 자연적인 순서라하면 알파벳, 숫자, 연대 와 같은 순서)
=> 정렬과 비교기능을 넣어야 한다면 Comparable을 고려하라는 것

- Comparable을 구현했다는 것 
== 그 클래스의 인스턴스들에는 자연적인 순서(natural order)가 있음을 뜻함
- 자연적인 순서(natural order): 일반적으로 받아들이는 대소 비교
ex) 1보다는 2가 큼
- Arrays.sort() 메서드는 자동으로 Comparable에 구현되어 있는 
compareTo() 메서드를 호출해서 사용함(String 클래스가 Comparable을 구현했기 때문에 
sort 하면 알파벳 순~)

## <구현 방법 2가지>
### 1) 단순비교
Comparable Interface를 사용하면 compareTo 매서드를 사용하고 박싱된 기본 타입 클래스가 제공하는 정적 compare 매서드 사용
### 2) 복잡한 비교
Comparator Interface가 제공하는 비교자 생성 매서드를 사용


## Comparator 사용 경우
- Comparable을 구현하지 않는 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용

## 작성법
- 동치인지를 비교하는 게 아니라 순서를 비교
- compareTo()에서 관계연산자 <, >를 사용하는 이전 방식은 거추장스럽게 오류를 유발하기 객체의 compare메서드를 사용
- 핵심 필드가 여러 개이면 어느 것을 먼저 비교하느냐에 따라 중요 
	=>  핵심적인 필드를 먼저 비교하자

## 주의사항
-  비교에 대한 기능은 만들어진 자바 클래스의 정적 매소드를 권장
- 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.
- compareTo 메서드에서 필드의 값을 비교할 때 < 와 > 연산자는 쓰지 말아야 한다. - 그 대신 박싱된 기본 타입 클래스가 제공하는 **정적 compare 메서드**나 Comparator 인터페이스가 제공하는 **비교자 생성 메서드**를 사용하자.
=> 두 값의 차이로 비교값을 사용하지 말자.

## 출처
effective-java 책 도서 

Comparator, Comparable : 
https://gmlwjd9405.github.io/2018/09/06/java-comparable-and-comparator.html

https://github.com/Meet-Coder-Study/book-effective-java

https://www.daleseo.com/java-comparable-comparator/