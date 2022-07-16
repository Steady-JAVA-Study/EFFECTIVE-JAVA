# ITEM 14. Comparable을 구현할지 고려하라

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻함

즉, 정렬과 비교기능을 넣어야 한다면 Comparable을 고려해야한다.

```java
public class Person {
    private int age;
    private String name;
    private double height;

}
```

위의 Person 클래스를 age를 1순위, height를 2순위, name을 3순위로 비교해야한다.

```java
public class Person implements Comparable<Person>{
    private int age;
    private String name;
    private double height;


    @Override
    public int compareTo(Person o) {
        int result = Integer.compare(this.age, o.age);
        if (result == 0) {
            result = Double.compare(this.height, o.height);
        }
        if (result == 0) {
            result = name.compareTo(o.name);
        }
        return result;
    }
}
```

### 비교자 생성 메서드를 활용한 비교자
```java

public class Person implements Comparable<Person>{
    private static final Comparator<Person> COMPARATOR =
            Comparator.comparing(Person::getAge)
                    .thenComparingDouble(Person::getHeight)
                    .thenComparing(person -> person.name);

    private int age;
    private String name;
    private double height;

    public int getAge() {
        return age;
    }

    public String getName() {
        return name;
    }

    public double getHeight() {
        return height;
    }

    @Override
    public int compareTo(Person o) {
        return COMPARATOR.compare(this, o);
    }
}
```

--- 

## Comparable vs. Comparator

### Comparable

- java.lang package
- 객체의 정렬기준을 정해줄 때 사용한다.
- Comparable 인터페이스의 구현체는 compareTo메서드를 구현해야 한다.

### Comparator

- java.util package
- 이미 정해진 정렬기준 외 다른 정렬기준을 사용하고 싶을때 사용한다.
- Comparator 인터페이스의 구현체는 compare메서드를 구현해야 한다.
- Comparator 인터페이스의 구현체는 그 자체가 정렬자로 사용된다. (정렬기준)
