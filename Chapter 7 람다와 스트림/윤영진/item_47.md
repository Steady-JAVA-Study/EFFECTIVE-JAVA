# ITEM 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

자바 7 이전에는 iterator 를 사용했다.

Stream 은 Iterable 을 확장하지 않는다.

즉 `for-each`로 stream 을 반복할 수 없다.

## Stream에서 Iterable 사이의 어댑터

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false)
}
```

위와 같은 어댑터는 비상시에만 만들어서 사용하자.

컬렉션으로 반환할 수 있는 상황이면 컬렉션으로 반환하자.

불가능할 때 Stream과 Iterable 중 자연스러운 것을 반환하자.