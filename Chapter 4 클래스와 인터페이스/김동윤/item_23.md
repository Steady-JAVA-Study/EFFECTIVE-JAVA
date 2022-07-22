# 태그 달린 클래스보다는 클래스 계층 구조 활용하라

## TAG ? 
> 클래스가 어떠한 타입인지에 대한 정보를 담고 있는 멤버 변수(필드)

## 태그 달린 클래스 단점
```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE}

    // 태그
    private final Shape shape;

    // 사각형일때만 사용
    private final double length;
    private final double width;

    // 원형일때만 사용
    private final double radius;

    public Figure(Shape shape, double radius) {
        this.shape = shape;
        this.length = 0;
        this.width = 0;
        this.radius = radius;
    }

    public Figure(Shape shape, double length, double width) {
        this.shape = shape;
        this.length = length;
        this.width = width;
        this.radius = 0;
    }

    public double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new IllegalArgumentException();
        }
    }
}
```
### 1. 코드 너무 혼잡
- 열거형 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드들 MANY
- ALSO 태그에 따른 메서드의 행동이 달라져야 하기 때문에 switch 문이 많아져 가독성 떨어지지
### 2. 메모리를 많이 사용
- 다른 행동을 위한 코드도 정의되기 때문에 메모리도 많이 사용

### 3. 불필요한 초기화가 필요하다.
- 필드를 final로 선언하려면 해당 태그에서 사용되지 않는 필드를 위해 생성자에서 불필요한 초기화가 필요

### 4. 새로운 코드(행동)를 추가하려면 태그를 추가
- 새로운 태그가 추가될 때마다 모든 switch 문에 새로운 태그에 대한 행동을 처리하는 코드를 추가

## 개선 방안

> 태그 달린 클래스를 클래스 계층구조로 변경
> 
#### 슈퍼타입(superset)
집합이 다른 집합의 모든 멤버를 포함
타입 정의가 다른 타입보다 일반적
#### 서브타입(subset)
집합에 포함되는 인스턴스들이 더 큰 집합에 포함
타입 정의가 다른 타입보다 좀 더 구체적


### 클래스 계층구조를 활용하는 서브타이핑(subTyping)
- 자바와 같은 객체 지향 언어는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단을 제공
- 태그 값에 따라 동작이 달라 지는 메서드들을 루트 클래스의 추상 메서드로 선언

> 1. 계층구조의 루트(root)가 될 추상 클래스 정의
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언
3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가
4. 하위 클래스에서 공통으로 사용하는 데이터 필드도 전부 루트클래스로 올리기

### 서브타이핑 PLUS
**서브타이핑(subtyping)** 
> - 타입 계층을 구성하기 위한 목적으로  사용
- 자식클래스와 부모 클래스의 행동이 호환되기 때문에 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체 가능 

- 행동이 호환되기 때문에 자식 클래스의 인스턴스가 부모클래스 인스턴스를 대체 가능
- 인터페이스 상속이라고도 불림 

- 타입 계층을 구성하기 위해 상속을 사용하는 경우 
- 자식 클래스와 부모 클래스의 행동이 호환되기 때문에 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 있음 
- 부모 클래스는 자식 클래스의 슈퍼타입이 되고 자식 클래스는 부모 클래스의 서브타입이 됨 
- 인터페이스 상속(interface inheritance) 이라고 부르기도 함


```java
public abstract class Figure { // 1. 루트(root)가 될 추상 클래스를 정의
    public abstract double area();
} // 2. 태그에 따라 행동이 달라지던 메서드는 추상 메서드로 구현(그렇지 않다면 일반 메서드)

public class Circle extends Figure { //자식 클래스 : 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() { //  구체 클래스에서 추상 메서드를 구현
        return Math.PI * (radius * radius);
    }
}

public class Rectangle extends Figure{ //자식 클래스
    private final double length;
    private final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    public double area() {
        return length * width;
    }
}

```



### GOOD POINT ?
- 컴파일 시에 형 검사(type checking)를 하기 용이
- 각 구체 클래스의 멤버 변수는 final로 선언 가능 

## 결론
> - 태그 달린 클래스를 써야 하는 상황은 거의 X
- 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애도 계층 구조로 대체하는 방법을
## 출처 
이펙티브 자바 도서 <br>
https://jjingho.tistory.com/66 <br>
ttps://gthoya.tistory.com/entry/서브클래싱과-서브타이핑 <br>
https://jaehun2841.github.io/2020/07/18/object-chapter13<br>
