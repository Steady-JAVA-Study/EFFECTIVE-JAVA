# ITEM 3. private 생성자나 열거타입으로 싱글턴임을 보증하라

> 싱글턴(singleton)?
> 
> 인스턴스를 오직 하나만 생성할 수 있는 클래스

## 싱글턴의 장점

한번의 객체 생성으로 재사용이 가능해져 메모리 낭비를 방지할 수 있다. 또한 전역성을 갖기 때문에 다른 객체와 공유가 용이하다.

## 싱글턴의 단점

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워질 수 있다. 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜 구현(Mock)으로 대체할 수 없기 때문이다.


## 싱글턴을 만드는 방식

### 1. public static final 필드 방식의 싱글턴
```java
public class Elvis {
    public static final Elvis INSATNCE = new Elvis();
    private Elvis() {...}
    
    public void leaveTheBuilding() {...}
}
```

#### 장점
1. 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
2. 간결하다.

### 2. 정적 팩터리 방식의 싱글턴

```java
public class Elvis {
    private static final Elvis INSATNCE = new Elvis();

    public static final Elvis getInstance() {
        return INSTANCE;
    }

    private Elvis() {...}
    
    public void leaveTheBuilding() {...}
}
```

Elvis 인스턴스는 getInstance()를 통해 항상 같은 객체의 참조를 반환한다.

#### 장점

1. (마음이 바뀌면) API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
2. 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.(아이템 30)
3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.(아이템 43, 44)

### 3. 열거 타입 방식의 싱글턴 

**가장 바람직한 방법**

```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { ... }
}
```

public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽하게 막아준다.
