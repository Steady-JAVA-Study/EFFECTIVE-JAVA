# ITEM 13. clone 재정의는 주의하여 진행하라

> Clonable 인터페이스
> 
> 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스
> 
> 믹스인: 다른 클래스의 부모클래스가 되지 않으면서 다른 클래스에서 사용할 수 있는 메서드를 포함하는 클래스, '상속'이 아닌 '포함'

```java
public interface Cloneable {
}
```

```java
java.lang.Object

protected native Object clone() throws CloneNotSupportedException;
```

Clonable 인터페이스는 빈 인터페이스이므로 Clonable을 구현했다고 clone()을 호출할 수 있는 것이 아니다.

clone() 메서드를 사용하기 위해서는 따로 오버라이딩을 해줘야한다. 또한, protected 이므로 같은 패키지에서만 접근 가능하다.

하지만 이를 포함한 여러 문제점에도 불구하고 Clonable 방식은 널리 쓰이고 있어서 잘 알아두는 것이 좋다.

## Clonable 인터페이스 무슨 일을 할까?

Object의 protected 메서드인 clone의 동작 방식을 결정한다.

Clonable을 구현한 클래스의 인터페이스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다.

**실무에서 Clonable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.**

## clone 메서드의 허술한 일반 규약

```text
1. x.clone() != x // 복사한 객체는 원본 객체와 독립적이다.
2. x.clone().getClass() == x.getClass() // 복사한 객체와 원본 객체는 같은 클래스이다.
3. x.clone().equals(x) 
```

![image](https://user-images.githubusercontent.com/83503188/179005002-97957871-e5df-46bd-8e28-8f97869442c5.png)

따라서 class를 확장할 때는 B에서도 clone을 오버라이딩 해야하는 경우가 생긴다.


---

배열 copy
```java
int[] a = {1,2,3,4};
int[] b = a; // shallow copy: 값만 동일한 다른 객체
b = a.clone(); // deep copy: 주소도 동일한 같은 객체
```

```java
Laptop[] a = {new Laptop("그램 16인치", "삼성")}
Laptop[] b = a.clone();
b[0].setCompany("LG");
```

Object의 reference value를 참조했기 때ㅜㅁㄴ에 `a[0] == b[0]` 즉 둘이 같은 객체를 가리키고 있다.

## 정리

```text
Primitive type의 배열이 아니면 쓰지 말자.
Copy Constructor or Copy Factory method를 활용하라
Clonable을 확장하지 마라

```

### 복사 생성자(Copy Constructor)
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(Stack s) {
        this.elements = s.elements.clone();
        this.size = s.size;
    }
}
```

### 복사 팩터리 메서드(Copy Factory Method)
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public static Stack newInstance(Stack s) {
        return new Stack(s.elements, s.size);
    }
}
```

