# ITEM 43. 람다보다는 메서드 참조를 사용하라

아이템 42에서 익명 클래스보다 람다를 사용하는 이유로 **간결함**을 설명하였는데, 람다보다 간결한 메서드 참조가 있다.

```java
public static void main(String[] args) throws IOException {

        List<Student> students = List.of(
                new Student("yoon1"),
                new Student("yoon2"),
                new Student("yoon3")
        );

        String names;
        // 일반 람다식
        names = students.stream().map(student -> student.getName()).collect(Collectors.joining(","));

        System.out.println(names);
        // 메서드 참조 표현식
        names = students.stream().map(Student::getName).collect(Collectors.joining(","));
        System.out.println(names);

    }

public static class Student {
    private String name;


    public Student(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }


}    
```

람다식이 단 하나의 메서드만 호출할 경우에 해당 람다식에서 불필요한 매개변수를 없앨 수 있다. 이때 형식은 `클래스::메서드명`의 형태로 써주면 간략한 메서드 참조로 변경할 수 있다.

## Map 인터페이스의 merge 메서드

Map 인터페이스의 다음과 같은 merge 메서드에서도 유용하게 사용 가능하다.

```java
default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction)
```

해당 메서드는 key 값을 받아 key가 map안에 없다면 value 값을 (key, value) 쌍으로 저장하고, key가 map안에 있다면 value 값을 BiFunction#apply 메서드에 적용해, (key, BiFunction#apply 메서드의 결과) 쌍으로 저장한다.

### 익명 클래스

```java
Map<String, Integer> map = new HashMap();

        map.merge("yoon4", 1, new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer oldValue, Integer newValue) {
                return oldValue + newValue;
            }
        });
        System.out.println(map); // 1

        map.merge("yoon4", 1, new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer oldValue, Integer newValue) {
                return oldValue + newValue;
            }
        });

        System.out.println(map); // 2
```

### 람다

```java
map.merge("yoon4", 1, (oldValue, newValue) -> oldValue + newValue);
```

### 메서드 참조
```java
map.merge("yoon4", 1, Integer::sum);
```

## 항상 메서드 참조가 정답은 아니다.

메서드 참조는 람다의 간단명료한 대안이 될 수 있다. **메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.**

```java
service.excute(GoshThisClassNameIsHumongous::action);
service.excute(() -> action);
```


