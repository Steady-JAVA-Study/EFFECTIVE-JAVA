# ITEM 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

> 실행자 ? 
> 
> 쉽게 말해 쓰레드를 만들고, 관리하는 것
> 
> 실행자(Executor) 는 task 단위로 작업을 하는데 하나의 task가 무조건 새로운 스레드를 사용하는 것이 아닌 내부적인 스레드 풀을 관리하며 수행



`java.util.concurrent` 패키지는 실행자 프레임워크라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.

```java
// 작업 큐를 생성하다. 
ExcutorService exec  = Executors.newSingleThreadExcutor();

// 다음은 이 Excutor에 실행할 태스크(task; 작업)를 넘기는 방법이다.
exec.execute(runnable);

// 다음은 Excutor를 우아하게 종료시키는 방법이다(이 작업이 실패하면 VM 자체가 종료되지 않을 것이다)
exec.shutdown();
```

### 실행자 서비스의 주요 기능

- 특정 태스크가 완료되기를 기다린다.
- 태스크 모음 중 아무것 하나(`invokeAny`) 혹은 모든 태스크(`invokeALL`)가 완료되기를 기다린다.
- 실행자 서비스가 종료하기를 기다린다(`awaitTermination`)
- 완료된 태스크들의 결과를 차례로 받는다(`ExecutorCompletionService`)
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(`ScheculedThreadPoolExecutor`)


### 그 외 기능

- 큐를 둘 이상의 스레드가 처리하게 하고 싶다면 간단히 다른 정적 팩터리를 이용하여 다른 종류의 실행자 서비스(스레드 풀)를 생성하면 된다.
- 평범하지 않은 실행자를 원한다면 ThreadPoolExecutor 클래스를 직접 사용해도 된다. 이 클래스로는 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수 있다.
- 작은 프로그램이나 가벼운 서버라면 Executors.newCachedThreadPool을 사용하라. 특별히 설정할 게 없고 일반적인 용도에 적합하게 동작한다.
- 무거운 프로덕션 서버에서는 스레드 개수를 고전한 Executors.newFixedThreadPool을 선택하거나 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 훨씬 낫다.

### ForkJoinTask

자바 7이 되면서 실행자 프레임워크는 포크-조인 태스크를 지원하도록 확장되었다. 포크-조인 태스크, 즉 ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 이고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다.

이러한 포크-조인 태스크를 직접 작성하고 튜닝하기란 어려운 일이지만, 포크-조인 풀을 이용해 만든 병렬 스트림을 이용하면 적은 노력으로 그 이점을 얻을 수 있다. 물론 포크-조인에 적합한 형태의 작업이어야 한다.


--- 

### Future

```java
public class ExecutorTest {

    public static void main(String[] args) {
        final int maxCore = Runtime.getRuntime().availableProcessors();
        System.out.println(maxCore);
        final ExecutorService executor = Executors.newFixedThreadPool(maxCore);
        final List<Future<String>> futures = new ArrayList<>();
        for (int i = 1; i < 5; i++) {
            final int index = i;
            futures.add(executor.submit(() -> {
                System.out.println("finished job " + index);
                return "job" + index + " " + Thread.currentThread().getName();
            }));
        }

        for (Future<String> future : futures) {
            String result = null;
            try {
                result = future.get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            System.out.println(result);
        }

        executor.shutdownNow();
        System.out.println("end");


    }
}
```

![image](https://user-images.githubusercontent.com/83503188/188879810-0c5e8608-dfee-4898-bfc1-5747e488a19d.png)


executor.submit()은 Future객체를 리턴

**Future**를 이용하면 예약된 작업에 대한 결과를 알 수 있다.

`future.get()`로 작업이 종료될 때 까지 기다린다. 따라서, 작업이 늦게 처리된다면 다른 작업에 대한 출력도 늦어진다. 

작업은 순서대로 처리되지 않을 수 있지만, 로그는 순차적으로 출력한다.


### BlockingQueue

```java
public class BlockingQueueTest {

    public static void main(String[] args) {

        ParallelExcutorService service = new ParallelExcutorService();
        service.submit("job1");
        service.submit("job2");
        service.submit("job3");
        service.submit("job4");

        for (int i = 0 ; i < 4; i++) {
            String result = service.take();
            System.out.println(result);
        }

        System.out.println("end");
        service.close();

    }

    private static class ParallelExcutorService {

        private final int maxCore = Runtime.getRuntime().availableProcessors();
        private final ExecutorService executor = Executors.newFixedThreadPool(maxCore);
        private final BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);

        public ParallelExcutorService() {
        }

        public void submit(String job) {
            executor.submit(() -> {
                String threadName = Thread.currentThread().getName();
                System.out.println("finished " + job);
                String result = job + ", " + threadName;
                try {
                    queue.put(result);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        public String take() {
            try {
                return queue.take();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new IllegalStateException(e);
            }
        }

        public void close() {
            List<Runnable> unfinishedTasks = executor.shutdownNow();
            if (!unfinishedTasks.isEmpty()) {
                System.out.println("Not all tasks finished before calling close: " + unfinishedTasks.size());
            }
        }
    }
}

```

Future 코드의 문제점인 작업이 늦게 처리된다면 다른 작업에 대한 출력도 늦어지는 문제를 BlockingQueue를 이용하면 해결할 수 있다.

작업이 끝날 때 BlockingQueue에 결과를 추가하고 메인쓰레드에서 Queue를 기다리면 된다.

전체적으로 보면, 멀티쓰레드에서 이 Queue에 add를 하는 구조이다. 동시성 문제가 발생할 것 같지만, BlockingQueue 객체는 동시성을 보장하도록 구현되어있다.


