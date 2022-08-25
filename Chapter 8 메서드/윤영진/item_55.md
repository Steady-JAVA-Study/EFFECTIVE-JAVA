# ITEM 55. 옵셔널 반환은 신중히 하라

## 메서드가 특정조건에서 값을 반환 할 수 없을때 사용했던 방법

- 자바 8 이전
  1. 예외 던지기
     1. 예외는 정말 예외적인 상황에서만 사용해야 한다.
     2. 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용이 비싸다.
  2. null을 반환
     1. 클라이언트에서 별도의 null 처리 코드를 추가해야 한다.
     2. null 처리를 무시하고 반환된 null은 NPE가 발생할 수 있다.

## Optional<T>

- 자바 8 이후 등장한 Optional을 통해 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있게 되었다.
- 아무것도 담지 않은 옵셔널 - '비었다', 어떤 값을 담은 옵셔널 - '비지 않았다'고 한다.
- 옵셔널은 원소를 최대 1개 가질 수 있는 '불변' 컬렉션이다.
- 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다.

### 예제 - 컬렉션에서 최댓값을 구한다.(컬렉션이 비어있으면 예외를 던진다.)

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("Empty collection");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```

Optional을 통한 리팩토링 - 컬렉션에서 최댓값을 구해 Optional로 반환한다.

```java
public static <E extends Comparable<E>>
Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}
```

위와 같이 빈 컬렉션을 건냈을 때 IllegalArgumentException을 던지는 것보다 Optional<E>를 반환하는 편이 더 낫다.

스트림의 종단 연산에서 Optional을 반환하는 방식 
```java
public static <E extends Comparable<E>>
Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

+ Optional.of(value)에 null을 넣으면 NPE가 발생하므로 null 값도 허용하는 옵셔널을 만들기 위해서는 Optional.ofNullable(value)를 사용하면 된다.

### 메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동

1. 기본 값을 정해둘 수 있다.
```text
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

2. 원하는 예외를 던질 수 있다.
```text
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

3. 항상 값이 채워져 있다고 가정한다. (NoSuchElementException을 주의한다)
```text
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```
- 옵셔널에 항상 값이 없다면 NoSuchElementException이 발생

--- 
## 기본 값 설정 비용이 클 경우

```text
public T orElse(T other)

public static String orElseBenchmark() {
    return Optional.of("baeldung").orElse(getRandomName());
}
```

```text
public T orElseGet(Supplier<? extends T> other)

public static String orElseGetBenchmark() {
    return Optional.of("baeldung").orElseGet(() -> getRandomName());
}
```

Supplier<T>를 인수로 받는 orElseGet을 사용하면 초기 설정 비용을 낮출 수 있다.

## isPresent

안전 벨브 역할의 메서드로, 옵셔널이 채워져 있으면 true를, 비어 있으면 false를 반환한다.

신중히 사용해야 하는데, 실제로 isPresent를 쓴 코드 중 상당수는 앞서 언급한 메서드들로 대체할 수 있으며, 그렇게 하면 더 짧고 명확하고 용법에 맞는 코드가 되기 때문이다.

```java
public class ParentPid {
    public static void main(String[] args) {
        ProcessHandle ph = ProcessHandle.current();

        // Inappropriate use of isPresent
        Optional<ProcessHandle> parentProcess = ph.parent();
        System.out.println("Parent PID: " + (parentProcess.isPresent() ?
                String.valueOf(parentProcess.get().pid()) : "N/A"));

        // Equivalent (and superior) code using orElse
        System.out.println("Parent PID: " +
            ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
    }
}
```

### Stream 사용할 경우

자바 9부터는 Optional을 stream으로 반환해주는 Optional.stream() 메서드가 추가되었다. 이 메서드는 Optioanl을 Stream으로 변환해주는 어댑터다. Optional에 값이 있으면 그 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 반환한다. 이를 Stream의 flatMap 메서드와 조합하면 아래와 같이 바꿀 수 있다.

```text
streamOfOptionals.flatMap(Optional::stream)
```

```text
streamOfOptionals
    .filter(Optional::isPresent)
    .map(Optional::get)

List<String> result = List.of(Optional.of("ABCD"), Optional.of("ABCC"), Optional.of("XVCD"))
                .stream().flatMap(Optional::stream)
                .filter(v -> v.startsWith("AB"))
                .collect(Collectors.toList());
    
List<String> result = Stream.of(Optional.of("ABCD"), Optional.of("ABCC"), Optional.of("XVCD")).flatMap(Optional::stream)
                .filter(v -> v.startsWith("AB"))
                .collect(Collectors.toList());
```



### Optional 주의사항

1. Optional.of(value)에 null을 넣으면 NPE를 발생하므로 주의해야 한다.

- null 값도 허용하는 Optional을 만들려면 Optional.ofNullable(value)를 사용하면 된다.

2. Optional을 반환하는 메서드에서는 절대 null을 반환하면 안 된다. 옵셔널을 도입한 취지를 완전히 무시하는 행위이다.

3. 컬렉션, 스트림, 배열, 옵셔널과 같은 컨테이너는 또 Optional로 감싸지 말아야 한다.
- 빈 Optional <List <T>>를 반환하기보다는 빈 List <T>를 반환하는 게 좋다.(아이템 54) 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다.

4. 반환 결과가 없을 수 있을 때, 클라이언트가 특별하게 처리해야 한다면 Optional을 반환한다.

5. Optional로 wrap 하고 다시 꺼내는 비용, 객체 생성 비용 등이 있으니 오버헤드가 발생한다. 유저가 실수할 여지를 줄이느냐, 성능을 중시하느냐 잘 고려해야 한다.

6. 박싱 된 기본 타입을 Optional로 반환하지 말자.
- 기본 타입을 감싸서 Optional로 또 감싼다면 2중으로 감싸게 된다. 성능상에 문제가 있다. 자바 API에는 int, double, long 전용 Optional 클래스를 만들었다.

7. Optional을 컬렉션의 Key, Value, 혹은 배열의 Element로 사용하지 말자

8. 필드를 Optional로 선언하는 건 대부분의 상황에서 좋지 않다.
- 대부분의 상황에서는 Optional을 사용하는 대신에 클래스를 나누어 필수 필드만을 담고 있는 클래스를 만들고 이를 확장해 선택 필드를 추가해야 한다는 일종의 신호이기도 하다.
