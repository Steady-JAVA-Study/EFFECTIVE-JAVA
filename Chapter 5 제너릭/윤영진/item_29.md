# ITEM 29. 이왕이면 제네릭 타입으로 만들라

### 예제

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

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
    
}
```

### 문제점

```java
public class StackTest {

    @Test
    public void StackTest() {
        MyStack stack = new MyStack();

        stack.push(1);
        stack.push("1");

        assertEquals(stack.pop().getClass(), String.class);
        assertEquals(stack.pop().getClass(), String.class); // Fail Test
    }
}
```

형변환을 클라이언트에서 해줘야 하며, 개발자가 실수한 경우 해당 에러는 컴파일 타임에 발생하지 않고 런타임에 발생해서 더욱 문제가 된다.

## 제네릭 타입의 Stack

```java
public class GenericStack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public GenericStack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // 에러 발생!
        // 실체화 불가 타입으로는 배열을 만들 수 없음.
    }

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
}
```

제네릭은 실체화 불가 타입이므로 배열 생성이 불가하다.

### 1. Object 배열로 생성하고 E 배열로 타입 캐스팅

```java
public GenericStack<E>() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

이제 컴파일에서 에러는 발생하지 않는 것을 알 수 있다.

하지만 unchecked cast warning 메시지를 볼 수 있다. 해당 메시지는 @SuppressWarnings("unchecked") 붙여서 숨길 수 있다.

책에서는 타입이 (일반적으로) 타입이 안전하지 않는 방법이라고 소개하고 있다.

### 2. Object 배열로 선언하고 생성하기

```java
public GenericStack<E>() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY]; 
    }

        ...
public E pop() {
        if (size == 0) {
        throw new EmptyStackException();
        }

        E result = (E) elements[--size];
        elements[size] = null;
        return result;
        }
```

컴파일 에러는 발생하지 않지만, 또 다른 경고 메시지가 출력이 되는 것을 볼 수 있다.

왜냐하면 E는 실체화 불가 타입이기 때문에 컴파일러는 런타임에 이루어지는 형변환이 안전한지 증명할 방법이 없기 때문이다.

마찬가지로 @SuppressWarnings("unchecked") 로 경고를 없앨 수 있다.

## 두 가지 방법 중 무엇을 사용?

책에서는 첫 번째 방법을 좀 더 선호한다고 말하고 있다. 하지만 힙 오염이라는 단점이 존재한다.

첫 번째 방법의 경우 가독성이 좋으며 캐스팅을 배열 생성 시 단 한 번만 해주면 된다는 장점을 가지고 있다.

## 결론 
클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.

그렇기 떄문에 새로운 타입을 설계할 때는 형변환 없이도 사용헐 수 있도록 만들어야 한다.

제네릭 타입을 사용하는 것은 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자들을 편하게 해주는 방법이다.

![image](https://user-images.githubusercontent.com/83503188/181278144-4edac4d9-6d1e-4477-8033-9483a9f4aaea.png)
