# ITEM 46. 스트림에서는 부작용 없는 함수를 사용하라

> Stream은 순수하여야 한다?
> 
> 순수 함수란 오직 입력만이 결과에 영향을 주는 함수, 다른 가변 상태를 참조하지 않으며, 함수는 다른 상태를 변경시키지 않아야 한다.

Stream의 각 변환의 단계는 이전 단계의 결과를 받아 처리하는 순수한 함수여야 한다.

즉, 다른 가변 상태를 참조하지 않으면서 함수 스스로도 다른 상태를 변경하면 안된다.

```java
public static Map<String, Long> streamSideEffect(final Stream<String> strings) {
    Map<String, Long> sideEffect = new HashMap<>();
    strings.forEach(string -> {
                sideEffect.merge(string, 1L, Long::sum);
            });
    return sideEffect;
}
```

현재 Stream 파이프라인은 `forEach` 종단 연산에서 sideEffect map의 상태를 변경하고 있다. `forEach` 를 사용하는 것은 Stream을 사용하는 것이 아닌 단순 반복문 사용에 불과하며 또 다른 SideEffect 를 발생시킨다.

```java
public class StreamLaziness {

    private static IntStream createAStreamAndPerformSomeSideEffectWithPeek() {
        return IntStream.of(1, 2, 3)
                .peek(number -> System.out.printf("First. My number is %d\n", number))
                .map(number -> number + 1);
    }

    private static void consumeTheStream(IntStream stream) {
        stream.filter(number -> number % 2 == 0)
                .forEach(number -> System.out.printf("Third. My number is %d\n", number));
    }

    public static void main(String[] args) {
        IntStream stream = createAStreamAndPerformSomeSideEffectWithPeek();
        System.out.println("Second. I should be the second group of prints");
        consumeTheStream(stream);
    }

}
```

```text
First. My number is 1
First. My number is 2
First. My number is 3
Second. I should be the second group of prints
Third. My number is 2
Third. My number is 4
```

위와 같은 결과를 예상하지만 실제로는,

```text
Second. I should be the second group of prints
First. My number is 1
Third. My number is 2
First. My number is 2
First. My number is 3
Third. My number is 4
```

중간 연산이라는 `peek` 메서드의 특성상 종단 연산 `forEach` 가 실행되는 시점에 Lazy 하게 Stream 파이프라인이 실행되는 것이라고 볼 수 있다.

```text
static void func1(){
    List<Integer> matched = new ArrayList<>();
    List<Integer> elements = new ArrayList<>();
    for(int i=0 ; i< 100 ; i++) {
        elements.add(i);
    }
    elements.parallelStream()
            .forEach(e -> {
                if(e>=50) {
                    System.out.println(Thread.currentThread().getId() +  " " + matched);
                    matched.add(e);
                }
            });
    System.out.println(matched.size());

}
```

```text
13 []
18 [null]
18 [81, 78]
.
.
.
17 [81, 78, 79, 80, 75, 76, 77, 84, 85, 86, 50, 51, 52, 68, 96, 97, 92, 87, 65, 66, 67, 88, 89, 62]
17 [81, 78, 79, 80, 75, 76, 77, 84, 85, 86, 50, 51, 52, 68, 96, 97, 92, 87, 65, 66, 67, 88, 89, 62, 63]
15 [81, 78, 79, 80, 75, 76, 77, 84, 85, 86, 50, 51, 52, 68, 96, 97, 92, 87, 65, 66, 67, 88, 89, 값62, 63, 64, 71]
```

쓰레드에 따라 값이 제가각일 뿐 아니라 추가되는 순서도 다르고 동시성을 보장하지 못하는 자료구조 ArrayList 에 값을 추가하다 보니 에러를 발생시킨다.

이럴때 Stream API의 부작용 없는 메서드를 사용하면 코드의 가독성 뿐 아니라 위와 같은 Side Effect를 해결할 수 있다.

```text
static void func2(){
    List<Integer> elements = new ArrayList<>();
    for(int i=0 ; i< 100 ; i++) {
        elements.add(i);
    }
    List<Integer> matched = elements.parallelStream()
            .filter(e -> e >= 50)
            .collect(toList()); // 가독성을 높이기 위해 Collectors를 static import
    System.out.println(matched.size() + "  " + matched);
}
```
```text
50  [50, 51, 52, 53, 54, 55, 56, 57, 58, 59, ... 98, 99]
```

바로 `collect()` 종단 연산이다. `collect()` 종단 연산은 Stream 연산을 객체 하나로 수집한다는 의미의 `Collectors` 클래스를 활요할 수 있다.


