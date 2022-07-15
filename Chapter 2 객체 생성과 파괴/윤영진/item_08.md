# ITEM 8. finalizer와 cleanser 사용을 피하라

자바는 두 가지 객체 소멸자를 제공한다. 그중 **finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.**

finalizer는 gc에 의해 호출된다.

```java
public class FinalizerExample {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("Clean up");

    }
    public void hello() {
        System.out.println("Hello");
    }

}
```

```java
public class SampleRunner {
    public static void main(String[] args) throws InterruptedException {
        SampleRunner runner = new SampleRunner();
        runner.run();

        Thread.sleep(1000l);

    }

    private void run() {
        // 아래 객체의 scope은 메소드 안 -> 객체는 유효성이 끝나므로 gc의 대상이 된다.
        // gc의 대상은 되지만 gc가 된다는 보장이 없으므로 finalizer가 호출되지 앟는다.
        // finalizer가 언제 호출된다는 보장을 못한다.
        
        FinalizerExample finalizerExample = new FinalizerExample();
        finalizerExample.hello();

    }
}
```

Java에서 자원 반납은 `finalizer`가 아닌 `try-with-resource` 또는 `try-finally`을 이용하자!  

자바 9에서는 finalizer를 사용 자제(deprecated) API로 지정하고 cleaner를 그 대안으로 소개했다. **cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.**



## 즉시 수행된다는 보장이 없다. 

`finalizer`와 `cleaner`는 즉시 수행된다는 보장이 없다. 언제 실행될 지 알 수 없으며 시간이 얼마나 걸릴지는 아무도 모른다. **즉 원하는 시점에 실행하게 하는 작업은 절대 할 수 없다.**

1. 파일 리소스를 반납하는 작업을 처리한다면 그 파일 리소스가 언제 처리 될지 알 수 없다.
2. 반납이 되지 않아 새로운 파일을 열지 못 하는 상황이 발생할 수 있다.
3. 동시에 열 수 있는 파일 개수가 제한되어 있다.

## 인스턴스 반납을 지연시킬 수 있다. 

Finalizer 쓰레드는 우선순위가 낮아서 언제 실행될지 모른다. 따라서 Finalizer 안에 어떤 작업이 있고, 그 작업을 쓰레드가 처리 못해서 대기하고 있다면, 해당 인스턴스는 GC가 되지 않고 계속 쌓이다가 결국엔 OutOfMemoryException이 발생할 수도 있다.

## 심각한 성능 문제도 있다.

`AutoClosable` 객체를 만들고, `try-with-resource`로 자원 반납을 하는데 걸리는 시간은 12ns 인데 반해, Finalizer를 사용한 경우에 550ns가 걸렸다. Cleaner를 사용할 경우에는 66ns가 걸렸다.

## Finalizer를 사용한 클래스는 Finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.

A라는 클래스가 존재하고 A클래스를 B클래스(나쁜놈)가 상속을 받음 -> B클래스가 인스턴스를 생성할 때 예외를 던진다. -> B클래스에서 finalizer를 오버라이딩 -> 예외를 던지므로 B가 생성되지 않아야 정상인데 finalizer는 반드시 실행되므로 인스턴스가 가진 메서드를 접근할 수 있다. 

이를 해결하기 위해서는 A클래스에서 finalizer를 fianl로 만들어 상속을 막아야 한다.

```java
@Override
protected final void finalize() throws Throwable {
        System.out.println("Clean up");

    }
```

## 자원 반납 하는 방법

자원 반납이 필요한 클래스는 `AutoClosable`을 구현해주고, `try-catch-resource`를 사용하거나, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.

```java
public class SampleResource implements AutoCloseable{
    @Override
    public void close() throws RuntimeException {
        System.out.println("close");
    }

    public void hello() {
        System.out.println("Hello");
    }
}
```

```java
public class SampleRunner {

    public static void main(String[] args) throws InterruptedException {
        SampleResource resource = null;
        try {
            resource = new SampleResource();
            resource.hello();
        } finally {
            if (resource != null) {
                resource.close();
            }
        }


    }

}
```

위의 방법을 아래와 같이 `try-with-resource`로 변경할 수 있다.

`AutoClosable`이 상속된 SampleResource의 사용이 다 끝난 경우 자동으로 close호출한다. 

**가장 이상적인 자원 반납**
```java
public class SampleRunner {

    public static void main(String[] args) throws InterruptedException {
        try (SampleResource resource = new SampleResource()) {
            resource.hello();
        }

    }

}

```

## Finalizer와 Cleaner를 안전망으로 쓰기

위의 코드와 같이 클라이언트에서 `try-with-resource` 또는 `try-finally`로 해줘야한다.

만약에 클라이언트가 클로징을 안했다는 가정하에 사용하자.

```java
public class SampleResource implements AutoCloseable{

    private boolean closed;

    @Override
    public void close() throws RuntimeException {
        if (this.closed) {
            throw new IllegalStateException();
        }
        closed = true;
        System.out.println("close");
    }

    public void hello() {
        System.out.println("Hello");
    }

    @Override
    protected void finalize() throws Throwable {
        if (!this.closed) close();
    }
}


```

## Cleaner를 안전망으로 활용하는 AutoClosable 클래스
```java
import java.lang.ref.Cleaner;

public class SampleResource implements AutoCloseable{

    private boolean closed;

    private static final Cleaner CLEANER = Cleaner.create();

    private final Cleaner.Cleanable cleanable;

    private final ResourceCleaner resourceCleaner;

    public SampleResource() {

        this.resourceCleaner = new ResourceCleaner();
        // 클리너 등록
        this.cleanable = CLEANER.register(this, resourceCleaner);

    }

    // 클리닝을 할 별도의 쓰레드
    private static class ResourceCleaner implements Runnable {

        // 정리할 Resource 멤버에 존재
        // SampleResource은 있으면 절대 안됨 순환참조 문제제
        @Override
        public void run() {
            System.out.println("Clean");
        }
    }
    @Override
    public void close() throws RuntimeException {
        if (this.closed) {
            throw new IllegalStateException();
        }
        closed = true;
        cleanable.clean();
        System.out.println("close");
    }

    public void hello() {
        System.out.println("Hello");
    }

}

```
