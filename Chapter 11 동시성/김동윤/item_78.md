# 공유 중인 가변 데이터는 동기화해 사용하라

## 동기화
> **동기화(Synchronized)**
: 멀티 스레드 환경에서 하나의 메서드나 블럭에 하나의 스레드만 접근하여 작업을 수행하도록 보장하는 것을 의미

## 동기화의 특징
- 한 객체가 일관된 상태를 가지고 생성되었을 때, 이 객체에 접근하는 메서드는 그 객체에 락(lock)을 검
- 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정 
- 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을 것 
- 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블럭에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게해줌

## synchronized 키워드
- 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장
- 많은 프로그래머가 동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다.
- 한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락(lock)을 검, 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정!
- 즉, 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킴, 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을 것
- 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수도 있음~


## 원자성
- 언어 명세상 long과 double 외의 변수를 읽고 쓰는 동작이 원자적이라 함
- 멀티 스레드 환경에서 변수를 동기화하지 않고 수정하더라도 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 의미
- 이말을 듣고 동기화하지 않아도 될거 같지만 이는 위험한 생각
   - 자바 언어 명세는 스레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 보이는가는 보장x

> 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요

### (ex) 동기화 안해서 무한루프 도는 코드
```java
public class Example {

    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(() -> {
           int i = 0;
           while (!stopRequested) {
               i++;
           }
        });

        thread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }

```
> - 아래의 예제코드를 보면 스레드가 시작되고 1초뒤에 stopRequested변수가 false로 바뀌면서 while문이 종료될거 같지만 무한 루프
- 해당 원인은 동기화에 있는데 동기화하지 않으면 메인 스레드에서 수정한 stopRequested 변수가 thread에서 언제쯤 보게될지 모르므로 무한 루프

## 해결방법
### Synchronized 사용

- 읽기(stopRequested), 쓰기(requestStop) 메서드를 동기화해서 만들어 제공함으로써 문제를 해결 가능
- 여기서 읽기/쓰기 메서드 모두 동기화를 처리했는데 만약 둘중 하나만하게 된다면 정확한 동작을 보장 X

```java
public class Example {

    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(() -> {
           int i = 0;
           while (stopRequested()) {
               i++;
           }
        });

        thread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

## 가변 데이터는 공유하지 말자

- 애초에 가변 데이터는 스레드 간 공유하지 않는게 the best~~
- 불변 데이터 정도만 공유를 하는게 제일 안전하고, 가변 데이터는 단일 스레드 환경에서만 사용하는 것이 good
- 혹은 한 스레드에서 데이터를 수정한 뒤 다른 스레드에 공유할 때 해당 객체에서 공유하는 부분만 동기화도 가능
- 이러면 다시 해당 객체를 수정하기 전까지 동기화 하지 않고 자유롭게 값을 동시에 읽어가도 ok
- 이러한 객체를 사실상 불변이라 하고, 이런 객체를 다른 스레드에 건네는 행위를 안전 발행이라 함

## reference
https://devfunny.tistory.com/669
https://kdg-is.tistory.com/401