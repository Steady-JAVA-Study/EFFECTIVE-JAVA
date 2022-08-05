# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
- enum 타입에는 extends 를 쓸 수 없다 !

## 열거타입도 확장할 수 있는 방법 有
>  **열거 타입이 인터페이스를 구현할 수 있다는 사실을 이용하는 것**

```java
public interface Operation {
    double apply(double x, double y);
}
```
________________________ 

```java
public enum BasicOperation implements Operation {

    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
}

```
- 열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.
## 다른 열거 타입 추가
```java
public enum ExtendedOperation implements Operation {

    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
}
```

## 결론
- 확장할 수 있는 열거 타입이 필요한 경우 인터페이스를 정의하여 구현 
- 열거 타입끼리는 상속 X
- 여러 열거 타입 간 공유하는 기능이 있으면, 클래스나 도우미 메서드로 분리!

## reference 
https://jaehun2841.github.io/2019/02/04/effective-java-item38/#%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%99%95%EC%9E%A5-%EA%B0%80%EB%8A%A5-%EC%97%B4%EA%B1%B0-%ED%83%80%EC%9E%85