# 스레드보다는 실행자, 태스크, 스트림을 애용하라

## java.util.concurrent 패키지의 등장
- java.util.concurrent 패키지는 실행자 프레임워크(Executor Framework)라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고있음
- 다음의 단 한줄로 작업 큐를 생성 ㅇㅇ

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class Test {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newSingleThreadExecutor();
    }
}
```
- 이 실행자에게 실행할 태스크(task; 작업)를 넘기는 방법
```java
exec.execute(runnable);
```
- 실행자를 종료
```java
exec.shutdown();
```

____________________
### 실행자 서비스의 주요 기능
- 특정 태스크가 완료되기를 기다린다.
- 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다리기
- 실행자 서비스가 종료하기를 기다리기(awaitTermination 메서드).
- 완료된 태스크들의 결과를 차례로 받기 (ExecutorCompletionService 이용).
- 태스크를 특정 시간에 혹은 주기적으로 실행 (ScheduledThreadpoolExecutor 이용).

_________________

### java.util.concurrent.Executors
- 큐를 둘 이상의 스레드가 처리하게 하고 싶다면 간단히 다른 정적 팩터리를 이용하여 다른 종류의 실행자 서비스(스레드 풀)를 생성
- 스레드 풀의 스레드 개수는 고정할 수도 있고 필요에 따라 늘어나거나 줄어들게 설정 가능
- 필요한 실행자 대부분은 java.util.concurrent.Executors의 정적 팩터리들을 이용하여 생성

### Executors.newCachedThreadPool
- 특별히 설정할게 없고 일반적인 용도에 적합하다. 무거운 프로뎍선 서버에는 좋지 X
- CachedThreadPool에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행, 가용한 스레드가 없다면 새로 하나를 생성
- 서버가 아주 무겁다면 CPU 이용률이 100%로 치닫고, 새로운 태스크가 도착하는 족족 또 다른 스레드를 생성하며 상황을 더욱 악화
- 따라서 무거운 프로덕션 서버에서는 스레드 개수를 고정한 Executors.newFixedThreadPool을 선택하거나 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 낫다

### 태스크
- 작업 큐를 손수 만드는 일은 삼가야하고, 스레드를 직접 다루는 것도 일반적으로 삼가야 함
- 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리
- 작업 단위를 나타내는 핵심 추상 개념이 태스크

#### 태스크의 두가지 
1) Runnable
2) Callable(값을 반환하고 임의의 예외를 던지기 가능)
- 태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행 정책을 선택할 수 있고, 생각이 바뀌면 언제든 변경가능 
- 핵심은 실행자 프레임워크가 작업 수행을 담당해준다는 것
