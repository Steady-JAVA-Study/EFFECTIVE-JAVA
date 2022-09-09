# 과도한 동기화는 피하라
- 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠질 수 있으며 예측할 수 없는 동작을 유발
## 응답불가와 안전실패를 피하려면 동기화 메서드나 블럭안에서는 제어권을 클라이언트에게 양도 X
 
> 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 X

## problem
### 교착상태 (Deadlock)
https://excited-hyun.tistory.com/121

> **교착상태의 발생 조건** - 모두 성립해야 발생
- **상호 배제** : 하나의 프로세스가 자원을 사용중일 때 다른 프로세스는 그를 사용할 수 없다.
- **점유 대기** : 최소 하나의 자원을 점유하고 있으면서 다른 프로세스가 사용중인 자원을 추가로 점유하기 위해 대기하는 프로세스가 존재한다.
- **비선점** : 다른 프로세스가 자원을 사용중인 경우 그 사용이 끝날 때 까지 강제로 뺏을 수 없다.
- **순환 대기** : 프로세스의 집합에서 순환형태로 자원을 대기하고 있어야 한다.


### 기아상태 (Starvation)
- 특정 프로세스의 우선 순위가 낮아서 원하는 자원을 계속 할당받지 못하는 상태
### 교착 상태와의 차이
- 교착상태는 프로세스가 자원을 얻지 못해 **다음 처리를 하지 못하는 상태**
- 기아 상태는 프로세스가 원하는 자원을 계속 할당 받지 못하는 상태


### 동기화 도중 제어권 넘기는 코드
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    // 관찰자 리스트
    private final List<SetObserver<E>> observers = new ArrayList<>();

    // 관찰자 추가 - 한 쓰레드만 접근 허용 가능 
    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }

    // 관찰자 제거
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    // Set add -> 관찰자의 added 메서드를 호출하여 동기화 도중 제어권을 넘기는 중
    private void notifyElementAdded(E element) {
        synchronized (observers) {
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
            result |= add(element);
        return result;
    }
}

@FunctionalInterface public interface SetObserver<E> {
    void added(ObservableSet<E> set, E element);
}
```
- Observer 패턴을 적용한 Set이다.
- 상태변화가 일어나면(add() 메서드 호출 시) 구독자들에게 해당 정보를 전파

### (1) 예외사항 :: 문제가 되는 코드 - `ConcurrentModificationException`

> ConcurrentModificationException
ConcurentModificationException은 해당 컬렉션 객체의 데이터 일관성에 대한 Exception으로서 컬렉션의 객체가 변함으로 인한 비정상적인 환경이 발생할 수 있다는 것을 알려주기 위한 Exception
https://m.blog.naver.com/tmondev/220393974518

```java
ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

set.addObserver(new SetObserver<>() {
    @Override
    public void added(ObservableSet<Integer> set, Integer element) {
        System.out.println(element);
        if (element == 23)
        //  리스트에서 원소를 제거할려는데 동시에 리스트를 순회하려 하기 때문
            set.removeObserver(this);
    }
});

for (int i = 0; i < 100; i++) {
    set.add(i);
}
```
- 위 코드는 추가된 값이 23일 경우 자기 자신을 구독해지 
- added() 메서드는 해당 관찰자를 동기화 중 실행되는 코드 
- removeObserver() 메서드는 관찰자 자기 자신을 콜백으로 넘겨 동기화 중인 관찰자를 제거하도록 동작하기 때문에 오류가 발생

### (2) 교착상태 :: 문제가 되는 코드  
- ConcurrentModificationException 은 피할 수 있지만 ,, 교착상태 ~ 

```java
ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
        //exec : 23 이면 새 스레드를 생성해서 관찰자 제거
        // 새 스레드에서 제거하는 거라서 concurrent 안발생
            ExecutorService exec =
                    Executors.newSingleThreadExecutor();
            try {
            // 여기서 lock이 발생하게 됩니다.(메인 스레드는 작업을 가리키고 있음) 
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
```
- 위 코드는 추가된 값이 23일 경우 새로운 스레드를 생성해서 관찰자를 제거
- 예외는 발생하지 않지만 메인 스레드가 관찰자에 대한 락을 가지고 있기 때문에 교착상태에 빠지게 됨
> 1 - try 문에 잇는 set.removeObserver 메서드를 호출하면 관찰자를 잠글려고 합니다.
2 - 하지만 락은 메인 메서드가 쥐고 있으므로 락을 얻을 수 없게됩니다.
3 - 이때 동시에 메인 스레드에서는 백그라운드 스레드가 관찰자를 제거하기를 기다리고 있습니다. 
4 - 교착상태에 빠지게 됩니다.
 
## 해결방법
### 콜백 메서드를 동기화 바깥으로 옮겨 해결
- 문제가 될 여지가 있는 코드를 동기화 블럭 바깥으로 옮겨 해결

```java

public void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized (observers) {
        snapshot = new ArrayList<>(observers);
    }

    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}

```

### 안전한 스레드를 위한 CopyOnWriteArrayList
https://jihyehwang09.github.io/2020/04/14/java-copyonwritearraylist/
**안전한 스레드를 위한 CopyOnWriteArrayList**

### 1) ArrayList와 synchronized를 이용한 동시성 제어
- synchronized 키워드는 해당되는 메서드나 블록을 한 번에 한 스레드씩 수행하도록 보장해주는 역할 

- 그러나 각각의 스레드가 과도하게 필요 이상으로 멈추거나 기다려야 하는 상황을 만듦

- 따라서 synchronized를 이용한 과도한 동기화는
성능을 떨어뜨릴 수 있고 교착 상태에 빠뜨릴 수 있으며, 예측할 수 없는 동작을 유발

### 2) CopyOnWriteArrayList
- CopyOnWriteArrayList는 ArrayList와는 달리,
List를 읽기 위해 어딘가에 전달할 때 원본이 아닌 복사본을 만들어서 전달

> - 자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList가 전확히 이 목적으로 특별히 설계, ArrayList를 구현한 클래스로, 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다. 
- 내부의 배열은 절대 수정되지 않으니 순회할때 락이 필요 없어 매우 빠르다.


```java

private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver (SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver (SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded (E element) {
    for (SetObserver<E> observer : observers)
    observer.added(this, element);
}

```

## 결론
- 클라이언트가 넘겨준 함수는 동기화 영역 안에서 호출 X
- 동기화 영역 안에서의 작업은 최소한의 작업만 수행 
- 가변 클래스를 설계할 때는 스스로 동기화가 필요한지 고민 
- 만약 내부에서 동기화 하도록 설계했다면 동기화 여부를 문서화

## reference
https://kdg-is.tistory.com/402?category=1031959
https://greeng00se.tistory.com/m/104