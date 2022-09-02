# 예외는 진짜 예외 상황에만 사용하라

```java
try {
    int i = 0;
    while(true) {
    	range[i++].climb();
    }
} catch (ArrayIndexOutOfBoundsException e) {
  //  무한루프를 돌다가 배열의 끝에 도달
}
```
- 배열의 각 요소마다 climb() 메서드를 호출하는구나, 배열의 범위 밖의 요소를 찾으면 예외가 발생하고 끝나는구나라고 생각할 수 ㅇㅇ
- 근데 이는 표준 관용구(for-loop)보다 훨씬 느림
`표준 관용구(for-loop) 사용 코드`
```java
//표준 관용구
for(Mountain m : range) {
  m.climb();
}
```

## 예외를 사용한 반복문의 오류
- 예외를 사용한 반복문은 코드를 헷갈리게 하고 성능을 떨어뜨리는데서 끝나지 않는다.
- 만약, 반복문 안에 버그가 숨어 있다면 흐름 제어에 쓰인 예외가 이 버그를 숨겨 디버깅을 훨씬 어렵게 할 것이다.
(ex) 반복문의 몸체에서 호출한 메서드 내부에서 반복문과 관련 없는 배열을 사용하다가 ArrayInexOutOfBoundsException을 일으킬 시,
표준 관용구였다면 이 버그는 예외를 잡지 않고 스택 추적 정보를 남긴 후 해당 스레드를 즉각 종료했을 것, 
BUT 예외를 통해 제어 흐름을 사용했다면, 버그 때문에 발생한 엉뚱한 예외를 정상적인 반복문에 의한 종료 상황으로 오해하고 넘어갈 것! 
=> 오히려 위험

##  예외 관점에서 잘 설계된 API란,
- 잘 설계된 API는 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 함
- 즉, '상태 의존적' 메소드를 제공하는 클래스는 '상태 검사' 메소드도 함께 제공해야 함
(ex) Iterator 인터페이스의 next(상태 의존적 메소드)와 hasNext(상태 검사 메소드)를 제공한다.
- 상태에 따라 좌지우지 되는 메소드를 사용하기 전에 상태 검사해주는 아이로 미리 사전 파악해놔야 한다는 말씀
```java
for(Iterator<Foo> i = collection.iterator(); i.hashNext();) {
  Foo foo = i.next();
  ...
}
```

- 만약 Iterator가 hasNext를 제공하지 않았으면 아래와 같이 장황하고 속도도 느리게 하고 심지어 버그까지 숨기는 아래의 코드를 써야해

```java
try{
  Iterator<Foo> i = collection.iterator();
  while(true) {
    Foo foo = i.next();
    ...
  }
} catch(NoSuchElementException e) {

}
```

## 결론
- 예외는 예외 상황에서만 사용 
- 정상적인 제어 흐름에서는 상태를 검사하여 개발하는 방법이 있어

### reference 
https://it-mesung.tistory.com/70