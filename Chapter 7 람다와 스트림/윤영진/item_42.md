# ITEM 42. 익명 클래스보다는 람다를 사용하라

## 익명 클래스

```java
public class Item42 {
    public static void main(String[] args) throws IOException {

        List<String> word = new ArrayList<>(List.of("ab", "Abc", "Aaa", "a"));

        Collections.sort(word, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return Integer.compare(o1.length(), o2.length());
            }
        });

        word.forEach(System.out::println);
    }
}
```

## 람다

```java
public class Item42 {
    public static void main(String[] args) throws IOException {

        List<String> word = new ArrayList<>(List.of("ab", "Abc", "Aaa", "a"));

        Collections.sort(word, (o1, o2) -> Integer.compare(o1.length(), o2.length()));

        word.forEach(System.out::println);
    }
}
```

자바 8부터 추상 메서드 하나만 가지는 인터페이스를 `함수형 인터페이스` 라고 부르며 특별하게 사용된다.

함수형 인터페이스의 인스턴스를 `람다식` 으로 만들 수 있다.

`람다` 는 익명 클래스와 비슷한 개념이지만 코드가 훨씬 간결하다.

람다식에서 타입을 생략한 이유는 컴파일러가 대신 인터페이스에 대한 타입추론을 대신 해주기 때문이다.

위의 코드에서 List<String>이 아니라 로 타입의 List 였다면 컴파일 오류가 발생한다.

### +Comparator

```java
Collections.sort(word, Comparator.comparingInt(String::length));
```

## 람다 활용 - 함수 인스턴스 필드 저장

```java
public enum Operation {
        PLUS("+", (x, y) -> x + y),
        MINUS("-", (x, y) -> x - y);

        private final String symbol;
        private final DoubleBinaryOperator op;

        Operation(String symbol, DoubleBinaryOperator op) {
            this.symbol = symbol;
            this.op = op;
        }

        public double apply(double x, double y) {
            return op.applyAsDouble(x, y);
        }

    }

```

```java
Operation op = Operation.PLUS;
double apply = op.apply(10, 20);
System.out.println(apply);
```





