# 이왕이면 제네릭 타입으로 만들라
- 클래스뿐만 아니라 메서드도 제네릭화 가능

`문제코드`
- 클라이언트에서 스택에서 꺼낸 객체를 형변환해야하고, 이는 런타임 오류로 이어질 가능성 => so 제네릭으로 바꾸는 것 good
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if(size == 0) throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
```
`제네릭 적용`
**제네릭으로 바꾸는 방법**
1) 클래스 선언에 타입 매개변수 추가
2) Object 를 적절한 타입 매개 변수로 바꾼다.

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // Error
    }
//E와 같은 실체화 불가한 타입으로는 배열을 만들 수 없다.
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        E result = elements[--size];
        elements[size] = null;
        return result;
    }
```

- E와 같은 실체화 불가한 타입으로는 배열을 만들 수 없다.

> 아이템28에서 배열보다는 리스트를 쓰라는 말이 있었다.
그러나 제네릭 타입 안에서 리스트를 사용하는게 항상 가능한것도 아니고 꼭 더 좋은것도 아니다.
결국 ArrayList 같은 제네릭 타입도 기본 타입인 배열을 사용하여 구현해야한다.

 
## 위 문제 해결 방안
1) 배열생성 금지제약을 우회하는 방법
- 이렇게 하면 오류는 해결할 수 있지만, 대신 경고

```java
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

- Unchecked cast 경고를 보여줍니다. → 타입이 안정적이지 않다는 것을 알려주고 있음


- 배열 elements는 private 필드에 저장되며, 클라이언트로 반환되거나 다른 메서드로 전달되는 일이  X
- push 메서드를 통해서만 배열에 데이터를 저장할 수 있으며, 저장되는 원소의 타입은 항상 E 
- 즉, 이 비검사 형변환은 안전
=> @SuppressWarnings 을 통해 해당 경고를 숨깁니다. [Item 27]

 ```java
 @SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

## 결론
- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전
- 새로운 타입을 설계할 땐 형변환 없이도 사용할 수 있도록 제네릭을 사용
- 기존 타입 중 제네릭이 있어야 하는 게 있다면, 제네릭 타입으로 변경
- 기존 클라이언트에게는 아무 영향을 주지 않으면서 새로운 사용자를 훨씬 편하게 해주지

## 출처 
https://codingwell.tistory.com/111?category=1013497
https://github.com/Meet-Coder-Study/book-effective-java/