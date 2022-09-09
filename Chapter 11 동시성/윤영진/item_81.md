# ITEM 81. wait와 notify보다는 동시성 유틸리티를 사용하라

## wait, notify

스레드의 상태 제어를 위한 메소드

`wait()`는 가지고 있던 고유 락을 해제하고, 스레드를 잠들게 하는 역할을 하는 메서드이고,

`notify()`는 잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 역할을 하는 메서드이다.

```java
public class ThreadB extends Thread{
	// 해당 쓰레드가 실행되면 자기 자신의 모니터링 락을 획득
   // 5번 반복하면서 0.5초씩 쉬면서 total에 값을 누적
   // 그후에 notify()메소드를 호출하여 wiat하고 있는 쓰레드를 깨움
    @Override
    public void run(){
        synchronized(this){
            for(int i=0; i<5 ; i++){
                System.out.println(i + "를 더합니다.");
                total += i;
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            notify(); // wait하고 있는 쓰레드를 깨움
        }
    }
}
```

```java
public class ThreadA {
    public static void main(String[] args){
        // 앞에서 만든 쓰레드 B를 만든 후 start 
        // 해당 쓰레드가 실행되면, 해당 쓰레드는 run메소드 안에서 자신의 모니터링 락을 획득
        ThreadB b = new ThreadB();
        b.start();

        // b에 대하여 동기화 블럭을 설정
        // 만약 main쓰레드가 아래의 블록을 위의 Thread보다 먼저 실행되었다면 wait를 하게 되면서 모니터링 락을 놓고 대기       
        synchronized(b){
            try{
                // b.wait()메소드를 호출.
                // 메인쓰레드는 정지
                // ThreadB가 5번 값을 더한 후 notify를 호출하게 되면 wait에서 깨어남
                System.out.println("b가 완료될때까지 기다립니다.");
                b.wait();
            }catch(InterruptedException e){
                e.printStackTrace();
            }

            //깨어난 후 결과를 출력
            System.out.println("Total is: " + b.total);
        }
    }
}
```
```text
b가 완료될때까지 기다립니다.
0를 더합니다.
1를 더합니다.
2를 더합니다.
3를 더합니다.
4를 더합니다.
Total is: 10
```

하지만, wait와 notify는 올바르게 사용하기가 아주 까다롭기 때문에, 대신 고수준 동시성 유틸리티를 사용하는 것이 좋다.

## 고수준 동시성 유틸리티

1. 실행자 프레임워크
2. 동시성 컬렉션(concurrent collection)
3. 동기화 장치(synchronizer)

### 동시성 컬렉션(concurrent collection)

표준 컬렉션(`List`, `Queue`, `Map`) + 동시성 = 동시성 컬렉션(`CopyOnWriteArrayList`, `ConcurrentHashMap`, `ConcurrentLinkedQueue`, ...)

동시성 컬렉션의 동시성을 무력화하는 것은 불가능하며, 외부에서 락을 걸면 오히려 속도가 느려진다. 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출할 수도 없다.

**상태 의존적 메서드**

위의 문제를 해결하기 위해 여러 기본 동작들을 하나로 묶는 상태 의존적 메서드가 추가되었다.

```java
putIfAbsent("key"); // 키에 매핑된 값이 없을 때에만 새 값을 집어넣고, 없으면 그 값을 반환한다.
```

**Collections.synchronized~**

Collections에서 제공해주는 `synchronized~()`를 사용하여 동기화한 컬렉션을 만드는 것 보다는 동시성 컬렉션을 사용하는 것이 성능적으로도 훨씬 좋다

### 동기화 장치(synchronizer)

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때 까지 기다리도록 확장되었다.

**대표적인 예시 - BlockingQueue**

```java
public interface BlockingQueue<E> extends Queue<E> {
		/**
     * Retrieves and removes the head of this queue, waiting if necessary
     * until an element becomes available.
     *
     * @return the head of this queue
     * @throws InterruptedException if interrupted while waiting
     */
    E take() throws InterruptedException;
}
```

BlockingQueue의 take()는 큐의 원소를 꺼내는 역할을 하는데, 만약 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다. ThreadPoolExcutor을 포함한 대부분의 실행자 서비스(아이템 80)에서 BlockingQueue를 사용한다.

#### 동기화 장치의 종류
- CountDownLatch, Semaphore
- CylicBarrier, Exchanger
- Phaser