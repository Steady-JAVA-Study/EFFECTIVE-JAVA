.# equals 는 일반 규약을 지지켜 재정의하라

> - equals는 논리적인 동치성을 확인하고 싶을 때 재정의 한다.
(ex) Enum: 값 클래스라고 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장할때

- 체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때 재정의를 해줘야 한다. 나머지 case 에서는 되도록 재정의하지 않는 것이 safe

## equals의 일반 규약
null이 아닌 모든 참조 값 x, y, z에 대해,

### 반사성 : 객체는 자기 자신과 같아야 함
```
x.equals(x) == true
```
### 대칭성 : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면, y.equals(x)가 true를 만족
```
x.equals(y) == true
-> y.equals(x) == true
```
### 추이성 : null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)가 true이면 x.equals(z)도 true
```
x.equals(y) == true
-> y.equals(z) == true
-> x.equals(z) == true
```
### 일관성 : x.equals(y)를 반복해도 값이 변하지 않는다.
### null-아님 : 묵시적 null 검사
```
x.equals(null) == false
```

- equals()를 재정의 시 타입 검사를 하게 된다면 지켜지게 된다. 

## equals 메서드 구현 방법 단계별 정리
### 1) == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 한다.
### 2) instanceof 연산자로 입력이 올바른 타입인지 확인한다.
위에서 설명했듯이 묵시적 null 체크를 할 수 있다.
또한 Collection interface처럼 특정 인터페이스의 타입 체크를 할 수도 있다.
### 3)입력을 올바른 타입으로 형변환한다.
instanceof 검사를 했기 때문에 100% 성공한다.
### 4) 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사

```java
@Override
public boolean equals(final Object o) {
    // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    if (this == o) {
        return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    if (!(o instanceof Point)) {
        return false;
    }

    // 3. 입력을 올바른 타입으로 형변환 한다.
    final Piece piece = (Piece) o;

    // 4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
    
    // float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
    return this.x == p.x && this.y == p.y;
    
    // 필드가 참조 차입이라면 equals를 사용한다.
    return str1.equals(str2);
    
    // null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
    return Objects.equals(Object, Object);
}
```

- float와 double 은 부동소수점이라서 compare to르 써야 한다.