# ITEM 76. 가능한 한 실패 원자적으로 만들라 

> 실패 원자적(failure-atomic)인 메서드
> 
> 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 하는 메서드

```java
public class PushPop {
    int size;
    int[] data;
    public PushPop() {
        size = 0;
        data = new int[50];
    }
    public void push(int input) {
        data[size++] = input;
    }
    public int pop() {
        return data[--size];
    }
}
```

위의 클래스를 생성하고 `pop()` 을 호출할 경우 예외가 발생하지만 size는 이미 1개가 줄어들어 원자성이 깨지게 된다.

## 메서드를 실패 원자적으로 만드는 방법 

1. 불변 객체로 설계하기 

메서드가 실패하면 새로운 객체가 만들어 지지 않을 수 있으나, 기존 객체가 불안정한 상태에 빠질 일은 없다. 왜냐하면 생성 시점에 고정되어 절대 변하지 않기 때문이다.

2. 가변 객체의 메서드의 경우 작업 수행 전 매개변수의 유효성을 검사 하기

객체의 내부 상태를 변경하기 전에 잠재적 예외의 가능성 대부분을 걸러낼 수 있는 방법이다.

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```

3. 객체의 임시 복사본에서 작업을 수행한 다음, 작업이 성공적으로 완료되면 원래 객체와 교체하는 것

데이터를 임시 자료구조에 저장해 작업하는 게 더 빠를 때 적용하기 좋은 방식

4. 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌리는 방법

주로 (디스크 기반의) 내구성(durability)을 보장해야 하는 자료구조에 쓰이는데, 자주 쓰이는 방법은 아니다.