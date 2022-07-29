# ITEM 31. 한정적 와일드카드를 사용해 API 유연성을 높여라

```java
public class Stack<E> {
    private final List<E> stack = new ArrayList<>();
    private int position = -1;
    
    public void push(E element) {
        position += 1;
        stack.add(element);
    }
        
    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }
    
    public E pop() {
        E popElement = stack.remove(position);
        position -= 1;
        return popElement;
    }

    public void popAll(Collection<E> dst) {
        dst.addAll(stack);
    }
    
    public boolean isEmpty() {
        return position == -1;
    }

		public int stackSize() {
        return position + 1;
    }
}
```

이런 Stack이 있다고 가정한다. 즉, 제네릭 E 타입의 값들을 관리하는 stack이다. 다만, 다음과 같이 컬랙션을 받아 stack에 넣는 pushAll 메서드를 추가한다고 가정한다.

위의 Stack 클래스의 pushAll() 메서드는 결함이 존재한다. Iterable의 원소 타입이 스택의 원소 타입과 일치하면 잘 작동하지만 타입이 다를 경우 매개변수화 타입이 불공변이기 때문에 컴파일 에러가 발생하게 된다.

### pushAll (producer - extends)

```java
@DisplayNmae("Number Stack에 Integer를 삽입이 가능한지")
@Test
void pushAll() {
    Stack<Number> stack = new Stack<>();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4);
    
    // Compile Error
    stack.pushAll(intList);
}
```

Number의 자식클래스인 Integer 또한 담고 싶은 경우?

논리적으로는 Number을 담을 수 있는 스택이 Integer도 담을 수 있어야 할 것 같으나 제네릭은 불공변이기 때문에 Integer가 Number의 하위타입인 것은 전혀 상관이 없어진다.

위의 문제를 한정적 와일드카드를 이용하면 해결할 수 있다.

```java
public class Stack<E> {
    private List<E> list = new ArrayList<>();

    public void pushAll(Iterable<? extends E> src) {
        for(E e : src) {
            push(e);
        }
    }

    public void push(E e) {
        list.add(e);
    }
}
```

`pushAll()` 입력 매개변수 타입은 E의 Iterable이 아니라 E의 하위타입의 Iterable 이라고 변경해주어 타입을 안전하고 깔끔하게 사용할 수 있게 된다.


### popAll (consumer - super)

```java
public void popAll(Collection<E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }    
}
```

위의 `popAll()` 또한 원소 타입이 일치한다면 문제없이 수행되지만 역시 Stack<Number>의 원소를 Object용 컬렉션으로 옮기려 한다고 하면, 결과는 앞에서 보았던것 처럼 Compile Error가 발생하게 된다.

```java
@DisplayName("Number stack을 Object 타입의 컬렉션으로 반환이 가능한지")
@Test
void popAll() {
    Stack<Number> stack = new Stack();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4);
    stack.pushAll(intList);
    
    List<Object> objList = new ArrayList<>();
    List<Number> numberList = new ArrayList<>();
    
    // Compile Error
    stack.popAll(objList);
    stack.popAll(numberList);
    
    assertThat(numberList).contains(1, 2, 3, 4);
}
```

위의 문제또한 와일드카드를 사용하면 해결할 수 있다.

```java
public void popAll(Collection<? super E> dst) {
    dst.addAll(list);
    list.clear();
}
```

## 타입 매개변수와 와일드카드

타입 매개변수와 와일드카드는 공통적인 부분이 있다. 때문에 메서드 정의시에 둘 중 어떤것을 사용해도 좋을 때가 있다.

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```
위 두 메서드는 둘다 큰 문제는 없다. 때문에 기본적으로 메서드 선언에 타입 매개변수가 한번만 등장한다면 와일드 카드로 대체한다. 즉, 위 두 swap에서는 두번째 swap이 적절한 것이다.

단, 두번째 `swap`은 다음과 같은 문제점이 있다.

```java
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```
위 코드는 컴파일 에러가 발생한다. 비한정적 와일드카드 List<?> 컬랙션은 null만 넣을 수 있기 때문이다. 이를 해결하기 위한 가장 좋은 방법은 실제 타입을 알려주는 도우미 메서드를 사용하는 것이다. 이때 도우미 메서드는 제네릭 메서드여야한다.

```java
public static void swap(List<?> list, int i, int j) {
	helper(list, i, j);
}

private static <E> void helper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```