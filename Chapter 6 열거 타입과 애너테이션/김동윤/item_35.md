# ordinal 메서드 대신 인스턴스 필드를 사용하라

- 대부분의 열거 타입 상수는 자연스럽게 인덱스(하나의 정수값) 가짐 
- So Java enum은 ordinal 이라는 메서드를 제공해 인덱스 반환 
- BUT ordinal은 사용하면 안됨

## ordinal을 잘못 사용한 예

- 대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응
- 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드 존재 
- 다음 코드는 합주단의 종류를 연주자가 1명인 솔로(solo)부터 10명인 디텍트(dectet)까지 정의한 열거 타입

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}

```
- 상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오동작하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.
- octet('8중주'라는 의미이다) 상수가 이미 있기 때문에 똑같이 8명이 연주하는 double quartet('복4중주')는 추가X, 즉 같은 숫자를 가질 수도 없어지게 되지~ 

- 또한, 값을 중간에 비워둘 수도 없다.
- 12명이 연주하는 3중 4중주 (triple quartet)를 추가한다고 가정
- 그러면 중간에 11명짜리 상수도 채워야 하는데, 11명으로 구성된 연주를 일컫는 이름 X
- 그래서 3중 4중주를 추가하려면 쓰이지 않는 더미(dummy) 상수를 같이 추가 필요
- 코드가 깔끔하지 못할 뿐 아니라, 쓰이지 않는 값이 많아질수록 실용성이 떨어짐

### SOLUTION : 열거 타입 상수에 연결된 값은 *ordinal* 메서드로 얻지말고, 인스턴스 필드에 저장! 

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}

```
___________________

> Enum 타입 사용할 때는 String으로 저장하자
enum 순서를 숫자로 데이터베이스에 저장 시, 항목이 변경,추가되면 숫자가 꼬임
→ 꼭 String 타입으로 저장
```java
// EnumType.ORDINAL: enum 순서를 숫자로 데이터베이스에 저장
// EnumType.STRING: enum 이름을 데이터베이스에 저장
@Enumerated(EnumType.STRING) //DB에 ENUM타입 매핑 시
private RoleType roleType;
```

(+) 
[깃허브 이펙티브 자바 출처 SAYS~](https://github.com/Meet-Coder-Study/book-effective-java/blob/main/6%EC%9E%A5/35_ordinal_%EB%A9%94%EC%84%9C%EB%93%9C_%EB%8C%80%EC%8B%A0_%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4_%ED%95%84%EB%93%9C%EB%A5%BC_%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%EA%B9%80%EC%84%B8%EC%9C%A4.md)
JPA는 Enum 필드도 테이블 컬럼과 매핑해줍니다.
그때 사용하는 어노테이션이 @Enumerated 입니다.
여기서 우리는 EnumType을 설정해줘야 하는데, ORDINAL과 STRING이 있습니다.
ORDINAL 타입은 방금 봤던 인덱스 기반의 값이며, STRING은 name() 메서드라고 생각하면 됩니다.
첫번째 문제점에서의 ZERO를 추가한 이후 그 도메인을 DB에서 불러올 경우 원래 ONE의 의미로 0을 저장하고 있던 필드는 ZERO가 나오게 될것입니다.
그렇다면 원래 있던 데이터들이 모두 꼬이게 될 것입니다.

### reference~
- Effective Java 3rd Edition
- https://rok93.tistory.com/