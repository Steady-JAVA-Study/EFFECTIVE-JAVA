# ITEM 52. 다중정의는 신중히 사용하라

> 다중정의?
> 
> 다중정의 == 오버로딩, 서로 다른 시그니처를 갖는 여러 메서드를 같은 이름으로 정의하는 것

## 오버로딩된 메서드들 중 어떤 메서드가 호출될지에 대한 선택은 컴파일 시점에 결정된다.


```java
public class Item52 {
    public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
        return "List";
    }
    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }
    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections) {
            System.out.println(classify(c));
        }
    }

}
```

```text
Unknown Collection
Unknown Collection
Unknown Collection
```

Collection<?> 타입의 배열을 순회한다면, 모든 배열의 원소를 Collection<?> 타입으로 인지한다.

왜냐하면 컴파일러는 Collection<?>[] 안에 항목이 HashSet 인지, ArrayList 인지는 런타임 시점에 알 수 있기 때문이다.

재정의된 메소드(Override)는 동적으로 선택 되지만 다중정의된 메소드(overloading)는 정적으로 선택이 된다.

## 오버라이드 메서드는 동적으로 선택된다.

```java
public class Item52 {

    static class Wine {
        String name() { return "wine"; }
    }

    static class SparklingWine extends Wine {
        @Override
        String name() { return "Sparkling Wine"; }
    }

    static class Champagne extends Wine {
        @Override String name() { return "Champagne"; }
    }

    public static void main(String[] args) {
        
        List<Wine> wineList = List.of(new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList) {
            System.out.println(wine.name());
        }
    }

}
```

```java
wine
Sparkling Wine
Champagne
```

## instance of

위의 오버로딩 예제를 정상적으로 동작시키기 위해서 instance of를 통해 타입 검사를 하면 된다.

```java
public static String classify(Collection<?> c) {
        return c instanceof Set ? "Set" :
                c instanceof List ? "List" : "Unknown Collection";
    }
```


## 다양한 메서드 이름으로 다중정의 사용 지양

- 같은 개수의 파라미터를 갖는 오버로딩 메서드 2개 이상을 갖지 마라

매개변수 개수가 같아야 한다면 오버로딩 대신 메서드 이름을 다르게 지어주는게 좋다

이러한 보수적 정책의 가장 좋은 예는 ObjectOutputStream 이다.

ObjectOutputStream은 write 메서드를 기본 자료형마다 오버로딩하지 않고 메서드에 자료형 이름을 붙였다. 예를 들어, writeInt(int) , writeBoolean(boolean), writeLong(long) 로 정의되어 있다.



