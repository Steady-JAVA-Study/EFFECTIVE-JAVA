# ITEM 44. 표준 함수형 인터페이스를 사용하라

> 함수형 인터페이스?
> 
> 오직 하나의 추상 메서드만 가지고 있는 인터페이스
 
## 항수형 인터페이스를 새로 정의하기 보다는 기본 함수형 인터페이스를 사용하자

```java
public class Item44 {
    public static void main(String[] args) throws IOException {
        Cache<String, Integer> cache = new Cache<>(3);

        cache.put("1", 1);
        cache.put("2", 2);
        cache.put("3", 3);
        cache.put("4", 4);

        System.out.println(String.join(", ", cache.keySet()));

    }
    public static class Cache<K, V> extends LinkedHashMap<K, V> {


        private final int limit;

        public Cache(int limit) {
            this.limit = limit;
        }

        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > this.limit;
        }
    }

}
```

```java
@FunctionalInterface
public interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}

public static class Cache<K, V> extends LinkedHashMap<K, V> {

    private final EldestEntryRemovalFunction<K, V> eldestEntryRemovalFunction;

    public Cache(EldestEntryRemovalFunction<K, V> eldestEntryRemovalFunction) {
        this.eldestEntryRemovalFunction = eldestEntryRemovalFunction;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return eldestEntryRemovalFunction.remove(this, eldest);
    }
}

Cache<String, Integer> cache = new Cache<>((map1, eldest) -> map1.size() > 3);


```

```java

public class Item44 {
    public static void main(String[] args) throws IOException {
        
        Cache<String, Integer> cache = new Cache<>((stringIntegerMap, stringIntegerEntry) -> stringIntegerMap.size() > 3);
        
        cache.put("1", 1);
        cache.put("2", 2);
        cache.put("3", 3);
        cache.put("4", 4);

        System.out.println(String.join(", ", cache.keySet()));

    }
    public static class Cache<K, V> extends LinkedHashMap<K, V> {

        private final BiPredicate<Map<K, V>, Map.Entry<K, V>> biPredicate;

        public Cache(BiPredicate<Map<K, V>, Map.Entry<K, V>> biPredicate) {
            this.biPredicate = biPredicate;
        }

        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return biPredicate.test(this, eldest);
        }
    }
    Cache<String, Integer> cache = new Cache<>((stringIntegerMap, stringIntegerEntry) -> stringIntegerMap.size() > 3);

}

```

## 기본 함수형 인터페이스

표준 함수형 인터페이스는 총 43개이며, 기본 인터페이스 6개를 통해서 나머지 부분을 유추할 수 있는 시스템이다.

<br>

### UnaryOperator (4개)

- 인수 타입, 반환 타입이 같다
- 인수 값이 1개이다.

| 인터페이스               | 메서드                                |
|---------------------|------------------------------------|
| UnaryOperator       | T apply(T t)                       |
| IntUnaryOperator    | int applyAsInt(int value)          |
| LongUnaryOperator   | long applyAsLong(long value)       |
| DoubleUnaryOperator | double applyAsDouble(double value) |

``` java
// 예시: String::toLowerCase
@Test
void UnaryOperator_Test() {
   UnaryOperator<String> unaryOperator = String::toLowerCase;
   assertThat(unaryOperator.apply("ABC")).isEqualTo("abc");
}
```

<br>

### BinaryOperator (4개)

- 인수 타입, 반환 타입이 같다
- 인수 값이 2개이다.

| 인터페이스                | 메서드                                               |
|----------------------|---------------------------------------------------|
| BinaryOperator       | T apply(T t1, T t2)                               |
| IntBinaryOperator    | int applyAsInt(int value1, int value2)            |
| LongBinaryOperator   | long applyAsLong(long value, long value2)         |
| DoubleBinaryOperator | double applyAsDouble(double value, double value2) |

``` java
// 예시: BigInteger::add
@Test
void BinaryOperator_Test() {
   BinaryOperator<BigInteger> binaryOperator = BigInteger::add;
   assertThat(binaryOperator.apply(BigInteger.ONE, BigInteger.TWO)).isEqualTo(BigInteger.valueOf(3L));
}
```

<br>

### Predicate (5개)

- 인수 값이 1개이며, boolean 반환한다.

| 인터페이스             | 메서드                        |
|-------------------|----------------------------|
| Predicate         | boolean test(T t)          |
| BiPredicate<T, U> | boolean test(T t, U u)     |
| IntPredicate      | boolean test(int value)    |
| LongPredicate     | boolean test(long value)   |
| DoublePredicate   | boolean test(double value) |

``` java
// 예시: Collection::isEmpty
@Test
void Predicate_Test() {
   Predicate<Collection> predicate = Collection::isEmpty;
   assertThat(predicate.test(new ArrayList<>())).isTrue();
}
```

<br>

### Function (17개)

- 인수 타입과 반환 타입이 다르다.

| 인터페이스               | 메서드                  |
|---------------------|----------------------|
| Function<T, R>      | R apply(T t)         |
| BiConsumer<T, U, R> | R apply(T t, U u)    |
| PFunction           | R apply(p value)     |
| PtoQFunction        | q applyAsQ(p value)  |
| ToPFunction         | p applyAsP(T t)      |
| ToPBiFunction<T, U> | p applyAsP(T t, U u) |

(P, Q 는 기본 자료형(Primitive Type) : Int, Long, Double)

``` java
// 예시: Arrays::asList
@Test
void Function_Test() {
   Function<int[], List> function = Arrays::asList;
   int[] array = {1, 2};
   List<Integer> list = function.apply(array);
   assertThat(function.apply(array).size()).isEqualTo(2);
}
```

<br>

### Supplier (5개)

- 예시: Instant::now
- 인수가 필요하지 않고, 반환이 있다.

| 인터페이스           | 메서드                    |
|-----------------|------------------------|
| Supplier        | T get()                |
| BooleanSupplier | boolean getAsBoolean() |
| IntSupplier     | int getAsInt()         |
| LongSupplier    | long getAsLong()       |
| DoubleSupplier  | double getAsDouble()   |

``` java
// 예시: Instant::now
@Test
void Supplier_Test() {
   Instant past = Instant.MIN;
   Supplier<Instant> supplier = Instant::now;
   assertThat(supplier.get().isAfter(past)).isTrue();
}
```

<br>

### Consumer (8개)

- 예시: System.out::println
- 인수가 필요하고, 반환이 없다.

| 인터페이스             | 메서드                            |
|-------------------|--------------------------------|
| Consumer          | void accept(T t)               |
| BiConsumer<T, U>  | void accept(T t, U u)          |
| IntConsumer       | void accept(int value)         |
| LongConsumer      | void accept(long value)        |
| DoubleConsumer    | void accept(double value)      |
| ObjIntConsumer    | void accept(T t, int value)    |
| ObjLongConsumer   | void accept(T t, long value)   |
| ObjDoubleConsumer | void accept(T t, double value) |

``` java
// 예시: System.out::println
@Test
void Consumer_Test() {
   OutputStream output = new ByteArrayOutputStream();
   System.setOut(new PrintStream(output));
   Consumer consumer = System.out::print;
   consumer.accept("안녕");
   assertThat(output.toString()).isEqualTo("안녕");
}
```

## 표준 함수형 인터페이스 대신 직접 구현해야하는 경우

```java
@FunctionalInterface
public interface Comparator<T> {
  int compare(T o1, T o2);
  //이하 생략
}
```

1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
2. 반드시 따라야 하는 규약이 있다.
3. 유용한 디폴트 메서드를 제공할 수 있다.



