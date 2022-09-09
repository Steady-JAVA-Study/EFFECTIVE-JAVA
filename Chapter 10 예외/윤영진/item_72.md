# ITEM 72. 표준 예외를 사용하라 

자바 라이브러리는 대부분 API 에서 쓰기에 충분한 수의 예외를 제공한다.

## 표준 예외를 재사용하면 얻는 장점

1. 다른 사람이 익히고 사용하기 쉬워지며, 가독성이 좋아진다.
2. 예외 클래스가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.

## 대표적인 예외

1. IllegalArgumentException 

호출자가 인수로 부적절한 값을 넘길 때 던지는 예외

ex. 양수를 넣어줘야 하는데 음수를 넣은 경우
```java
public class MainRunner {
  
    List<Integer> list = new ArrayList<>();
  
    public void positiveAdd(Integer number) {
        if (number < 0) {
            throw new IllegalArgumentException("음수값은 허용하지 않음");
        }
        list.add(number);
    }

    public static void main(String[] args) {
        MainRunner mainRunner = new MainRunner();
        mainRunner.positiveAdd(-1);
    }
}
```

2. IllegalStateException

대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 주로 사용한다.

```java
public class MainRunner {

    List<Integer> list = new ArrayList<>();
    
    public void evenIndexAdd(Integer number){
        if(list.size()%2 != 0){
            throw new IllegalStateException("짝수 인덱스에 들어갈 준비가 되지 않음");
        }
        list.add(number);
    }

    public static void main(String[] args) {
        MainRunner mainRunner = new MainRunner();
        mainRunner.evenIndexAdd(0);
        mainRunner.evenIndexAdd(1);
    }
}
```

3. NullPointerException

null 값을 허용하지 않는 메서드에 null 을 건네는 경우 사용한다.

```java
public class MainRunner {

    List<Integer> list = new ArrayList<>();

    public void add(Integer number){
        if(Objects.isNull(number)){
            throw new NullPointerException("값이 null 입니다.");
        }
        list.add(number);
    }

    public static void main(String[] args) {
        MainRunner mainRunner = new MainRunner();
        mainRunner.add(null);
    }
}
```


4. IndexOutOfBoundsException

시퀀스의 허용범위를 넘는 값을 건널때 사용한다.

```java
public class MainRunner {

    List<Integer> list = new ArrayList<>();

    public void add(Integer number){
        if(list.size()>3){
            throw new IndexOutOfBoundsException("요소를 3개 이상 설정 할 수 없습니다.");
        }
        list.add(number);
    }

    public static void main(String[] args) {
        MainRunner mainRunner = new MainRunner();
        mainRunner.add(1);
        mainRunner.add(2);
        mainRunner.add(3);
    }
}
```

5. ConcurrentModificationException

단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때 사용한다. 

6. UnsupportedOperationException

클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 던진다.

대부분 객체는 자신이 정의한 메서드를 모두 지원하니 흔히 쓰이는 예외는 아니다.


