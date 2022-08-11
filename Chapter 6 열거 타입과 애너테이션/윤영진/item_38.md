# ITEM 37. ordinal 인덱싱 대신 EnumMap을 사용하라

## EnumMap 

Map 인터페이스에서 키를 특정 열거형 타입만을 사용하도록 하는 구현체

```java
@AllArgsConstructor
public class Plant {
    // 식물의 생애 주기를 관리하는 열거 타입
    enum LifeCycle {
        ANNUAL, // 한해살이
        PERENNIAL, // 여러해살이
        BIENNIAL // 두해살이
    }

    final String name;
    final LifeCycle lifeCycle;

    @Override
    public String toString() {
        return name;
    }
}
```

**요구사항**: 식물들을 배열 하나로 관리, 생애주기로 묶어야 한다.

### ordinal 인덱싱

```java
public void addPlant(List<Plant> garden) {
        // 생애주기 3가지로 만들어지는 Set
        Set<Plant>[] plantsByLifeCycle = new Set[Plant.LifeCycle.values().length];

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
        plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant p : garden) {
        plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
        }

}
```

많은 단점을 가지고 있으므로 EnumMap을 사용한다.

```java
public void addPlantTypeEnumMap(List<Plant> garden) {
        Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

        for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
            plantByLifeCycle.put(lc, new HashSet<>());
        }

        for (Plant p : garden) {
            plantByLifeCycle.get(p.lifeCycle).add(p);
        }

}
```



