# ITEM 83. 지연 초기화는 신중히 사용하라

> 지연 초기화(lazy initialization)
> 
> 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법, 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는다.

지연 초기화는 **최적화 용도**로 사용된다.

클래스와 인스턴스 초기화 때 발생하는 **위험한 순환 문제**를 해결하는 효과도 있다. 

![image](https://user-images.githubusercontent.com/83503188/189125783-c7286b0d-ae9e-4af1-a316-d1855052c188.png)


```java
public class Speaker {
    private static Speaker instance;
    private Speaker() {}
    
    public static synchronized Speaker getInstance() {
        if(instance == null) {
            instance = new Speaker();
        }
        return instance;
    }
}
```

대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

### 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사(double-check) 관용구를 사용하라.

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null) {  // 첫 번째 검사(락 사용 안 함)
        return result;
    }
    
    synchronized(this) {
        if (field == null) {  // 두 번째 검사(락 사용)
            field = computeFieldValue();
        }
        return field;
    }
}
```