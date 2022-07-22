# ITEM 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 접근자(getter)와 변경자(setter) 메서드를 활용한 데이터 캡슐화

패키지 바깥에서 접글할 수 있는 클래스라면 접근자를 제공하여 내부 표현 방식을 언제든 바꿀 수 있도록 유연성을 얻을 수 있다.

package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다. ->  package-private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}

```



