# 한정적 와일드카드를 사용해 API 유연성을 높이라

- List가 공변이면 List은 List의 하위 타입
- 불공변은 하위 타입이 성립하지 않음
- BUT 이제 배울 매개변수화 타입(제네릭)은 List가 List의 역할을 제대로 수행하지 못하기 때문에 리소코프 치환 원칙에 어긋나 불공변이라고 한다.
- 즉 아래와 같이 사용하지 못한다는 것이다.
```java 
ArrayList<String> strings = new ArrayList<Object>();
ArrayList<Object> objects = new ArrayList<String>();
```

**제네릭에서 불공변의 문제점**
  
```java 
package com.spring.test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.EmptyStackException;
import java.util.List;

public class Stack<E> {
    private List<E> elements;
    private int size = 0;

    public Stack() {
        this.elements = new ArrayList<>();
    }

    public static void main(String[] args) {
        Stack<Number> stack = new Stack<>();
        Iterable<Integer> integers = Arrays.asList(1);
        stack.pushAll(integers);
    }

    public void push(E o) {
        elements.add(o);
        size++;
    }

    public <E> E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E element = (E)elements.get(size);
        elements.remove(size--);
        return element;
    }

    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }

    public boolean isEmpty() {
        return elements.size() == 0;
    }
}
```
- 하위 타입이여서 논리적으로 잘 동작할 것 같지만 불공변이기 때문에 아래와 같이 에러가 발생
- 따라서 자바는 한정적 와일드카드 타입을 지원

## 한정적 와일드 카드 타입을 사용할 때
- 원소의 생산자나 소비자용 입력 매개변수 일 때
- 원소가 생산자와 소비자 역할을 동시에 사용하지 말 것, 이는 타입을 정확히 지정해야하는 상황 
- 유연성을 극대화


## bounded Wlidcard vs unbounded Wlidcard
### 한정적 와일드카드
- 특정 타입을 제한
- (ex) 
```
<T extends Number>
```
### 비한장적 와일드카드
- ? 와 같은 타입으로 어떤 타입이 오던 관계가 없다는 뜻
