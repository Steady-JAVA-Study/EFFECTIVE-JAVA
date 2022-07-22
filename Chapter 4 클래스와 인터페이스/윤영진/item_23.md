# ITEM 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

> 태그 달린 클래스?
> 
> 두 가지 이상의 의미를 표현할 수 있으며, 그 중 현재 표현하는 의미를 태그 값으로 알려주는 클래스를 태그 달린 클래스라고 한다.

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

## 단점

1. 쓸데없는 코드가 많아 가독성이 안좋다.
2. 만약 필드를 final로 선언하고 싶으면 굳이 사용하지 않는 값도 생성자에서 초기화해야 한다.
3. 또 다른 도형을 추가하려면 쓸데없는 코드가 늘어난다.

## 계층구조를 사용하자.

```java
public abstract class Figure {
    public abstract double area();
}

public class Circle extends Figure {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * (radius * radius);
    }
}

public class Rectangle extends Figure{
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

1. 계층구조의 루트(root)가 될 추상 클래스를 정의한다.
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
4. 하위 클래스에서 공통으로 사용하는 데이터 필드도 전부 루트클래스로 올린다.