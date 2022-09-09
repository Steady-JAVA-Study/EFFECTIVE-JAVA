# ITEM 78. 공유 중인 가변 데이터는 동기화해 사용하라


### 배타적 실행

한 스레드가 변경하는 중이라면 다른 스레드가 보지 못하게 막는 용도를 말한다.

즉, Lock 을 걸어 동시에 여러 스레드가 접근하는 것을 차단한다.

### Synchronized

> 동기화(synchronized) 키워드
>
> 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.

```java
public class ThreadCounter {
    public static int count = 0;

    public static void threadRunner() throws InterruptedException {
        int count = ThreadCounter.count;
        TimeUnit.SECONDS.sleep(2);
        ThreadCounter.count = count + 1;
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 5; i++) {
            int finalI = i;

            Thread backgroundThread = new Thread(() -> {
                try {
                    threadRunner();
                } catch (InterruptedException e) {
                    e.printStackTrace();

                }
            });
            backgroundThread.start();
        }
        TimeUnit.SECONDS.sleep(10);
        System.out.println("end = " + count);
    }
}
```

![image](https://user-images.githubusercontent.com/83503188/188855520-1a57d63d-a7ba-46b0-a2cc-855f0df17a3a.png)

위의 `threadRunner()` 에  Synchronized 키워드를 붙여주면

```java
public synchronized static void threadRunner() throws InterruptedException {
        int count = ThreadCounter.count;
        TimeUnit.SECONDS.sleep(2);
        ThreadCounter.count = count + 1;
    }
```

![image](https://user-images.githubusercontent.com/83503188/188856416-29b0ace7-40cf-48cc-b632-bf127e458500.png)

단, 동기화가 필요한 코드수를 최소한으로 줄여야지 최적화된 프로그램이라고 할 수 있다.

### Volatile

> Volatile 
> 
> Volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

Volatile은 multi-read, one-write 상황일 때 유용하다.

```java
public class StopThreadVolatile {
    
    private static volatile boolean stopRequested;
    
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
}
```

#### Volatile 원리

Volatile 키워드를 사용하면 데이터를 메인 메모리에 저장하게된다.

Multi-thread 환경에서는 task를 수행하는 동안 성능 향상을 위해 변수 데이터들을 Main memory에서 활용하는 것이 아닌 CPU cache에 저장하고 활용한다.

처음 예제에서 count에 Volatile 키워드를 통해 문제를 해결할 수 없었던 이유는 count를 가져와서 가져온 값을 1씩 증가시키기 때문이다. 

![image](https://user-images.githubusercontent.com/83503188/188857944-9a85da53-9617-4f46-a157-a59a17d1686b.png)
