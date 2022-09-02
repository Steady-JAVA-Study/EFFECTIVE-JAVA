# ITEM 69. 예외는 진짜 예외 상황에만 사용하라

예외는 오직 예외 상황에서만 써야한다. 절대 일상적인 제어 흐름용으로 사용해선 안 된다.

```java
try {
  int i = 0;
  while(true) {
    range[i++].climb();
  }
} catch (ArrayIndexOutOfBoundsException e) {
}
```

위 코드의 문제

1. 코드가 지저분하다.
2. 반복문 내부에서 `ArrayIndexOutOfBoundsException` 가 아닌 다른 예외가 발생했을 때 문제없이 넘어간다.


## 잘 설계된 API는 정상적인 흐름에서 예외를 사용하지 않는다.

```java
//=============== 
// 상태 검사 메서드와 상태 의존메서드를 제공
    public boolean hasEngine() {
        return engine != null;
    }

    public Point move() {
        return new Point(engine.power());
    }


//=================
// 상태 검사 메서드를 제공하지 않고 정상적인 값을 반환할 수 없다면 특정 값을 제공
    public Point moveValue() {
        if (engine == null) {
            return new Point(0);
        }
        return new Point(engine.power());
    }
//===============
// optional 제공
    public Optional<Point> moveOptional() {
        if (engine == null) {
            return Optional.empty();
        }
        return Optional.of(new Point(engine.power()));
    }
```

1. 상태 검사메서드와 의존 메서드를 제공하는 방식
2. 올바르지 않은 상태일때 특정 값을 제공
3. 올바르지 않은 상태를 대비해 optional 제공

위 세가지 방식으로 해당 api에서 올바르지 않은 상태일때 exception을 발생시키지 않는 방법이다.

여기서 2,3번째 방식은 의존메서드에 상태 검사가 중복으로 들어갈때 사용하거나, 멀티 스레드 환경에서 상태 검사 후 의존메서드를 부르는 찰나에 원하지 않는 결과가 발생할 수 있으므로 해당 상황에 대비하여 사용하는 것이 좋다.