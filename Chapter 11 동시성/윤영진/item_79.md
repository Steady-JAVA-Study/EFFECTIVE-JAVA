# ITEM 79. 과도한 동기화는 피하라

과도한 동기화는 
1. 성능을 떨어뜨리고
2. 교착상태에 빠뜨리고
3. 예측할 수 없는 동작을
일으킨다.

응답 불가와 안전 실패(safety failure) 를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안된다.

> 응답 불가
> 
> Exception, Timeout, ...

> 안전 실패
> 
> 실패 시에도 작업을 중단하지 않는다.

위와 같은 메서드를 '무슨 일을 할지 알지 못하며 통제할 수 없는' 외계인 메서드이다.

외계인 메서드를 사용하면 동기화된 영역은 
1. 예외를 일으키거나
2. 교착상태에 빠지거나
3. 데이터를 훼손
할 수 있다.

### 예시

어떤 집합(Set)을 감싼 래퍼 클래스이고 집합에 원소가 추가되면 알림을 받는 관찰자 패턴을 사용한 예제 코드이다.

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // notifyElementAdded 호출
        return result;
    }
}
```

관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두 메서드 모두 SetObserver 콜백 인터페이스의 인스턴스를 매개변수로 갖는다.

#### 문제를 일으킬 수 있는 익명함수 콜백 - 예외 발생

```java
public class Test {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) // 값이 23이면 자신을 구독해지
                    s.removeObserver(this); // this를 넘겨주어야하기 때문에 람다가 아닌 익명 클래스 사용
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

- 이 프로그램은 23까지 출력한 다음 ConcurrentModificationException 예외를 던진다
- 관찰자의 added 메서드를 호출한 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다.
- 리스트에서 원소를 제거하려 하는데, 마침 지금 이 리스트를 순회하는 도중이다. 즉, 허용되지 않은 동작이다.

#### 문제를 일으키는 스레드 - 교착상태

> Deadlock (교착상태)
> 
> **필요조건** 

> 상호배제
> 
> 점유대기
> 
> 비선점
> 
> 순환대기

![image](https://user-images.githubusercontent.com/83503188/188866825-97108179-7e5d-402f-828a-eb4656878ed7.png)

```java
public class Test {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<Integer>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    ExecutorService exec = Executors.newSingleThreadExecutor();
                    try {
                        // 여기서 lock이 발생한다. (메인 스레드는 작업을 기리고 있음)
                        // 특정 태스크가 완료되기를 기다린다. (submit의 get 메서드)
                        exec.submit(() -> s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
- 위의 exec는 새로운 쓰레드에 removeObserver를 수행하는 람다함수를 넘긴다.
- 이 메서드는 실행하면 23까지 출력하고 계속 멈춰있다.(교착상태)
