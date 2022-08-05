# ordinal 인덱싱 대신 EnumMap을 사용하라

> **ordinal**
해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 메소드

## ordinal()을 배열 인덱스로 사용  X
- 식물의 생애주기를 열거 타입으로 표현한 LifeCycle 열거 타입
```java
class Plant {
    enum Lifecycle { ANNUAL, PERENNIAL, BIENNIAL }
    final String name;
    final Lifecycle lifeCycle;
    
    Plant(String name, Lifecycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    } 
}
```
- Plant 클래스는 LifeCycle 열거 타입을 멤버 변수로 가지고 있다. 이를 이용해서 식물들을 생애주기로 묶어보기 
```java

// 1
Set<Plant>[] plantsByLifeCycle = 
(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

// 2 
for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p); 
    //이런 코드 XXX, ordinal 로 인덱싱 
}

// 3 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

- 1. Set 배열을 생성해 생애주기별로 관리,  총 3개의 배열이 만들어질 것,  각 배열을 순회하여 빈 HashSet으로 초기화
- 2. plant들을 배열의 Set에 추가, 이때 plant가 가지고있는 LifeCycle 열거타입의 ordinal 값으로 배열의 인덱스를 결정, 그 결과 식물의 생애주기 별로 Set에 추가
- 3. 열거 타입의 values로 반환되는 열거 타입 상수 배열의 순서는 ordinal 값으로 결정 => Set 배열의 각 Set이 의미하는 생애주기는 values의 순서와 the same

### **Problem**

- `Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length]`
배열은 제네릭과 호환 X (비검사 형변환) 이 수행, 컴파일 X)
- ( 배열은 공변, 제네릭은 불공변)
- ( 배열은 런타임에 실체화, 제네릭 타입은 런타임에 소거 )
=> 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행
```
ENUMCLASS.values()
=> ENUM 안의 상수들 모두 델꼬오는 것 
```
- 사실상 배열은 각 인덱스가 의미하는 바를 알지못하기 때문에 출력 결과에 직접 레이블을 달아야 한다.
// [[],[] ... ] 
- 정수는 열거 타입과 달리 타입 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 직접 보증 필요

## ENUM MAP 을 사용하자 ~ 
- key 값 존재 

> EnumMap
EnumMap은 열거 타입을 키로 사용하는 Map 구현체
EnumMap 사용법
간단한 Enum
```
enum DayOfWeek {
    MON, TUE, WED, THU, FRI, SAT, SUN
}
```
EnumMap 생성 코드
```
Map<DayOfWeek, String> scheduleMap = new EnumMap<>(DayOfWeek.class);
```
EnumMap은 AbstractMap의 구현체이기 때문에, 일반적으로 알고있는 Map의 기능들을 모두 사용 할 수 있다.
```
scheduleMap.put(DayOfWeek.MON, "Study");
scheduleMap.get(DayOfWeek.MON);
```

- 위 예제에서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 역할 
=> 따라서 Map을 사용 OK ~ 
- EnumMap은 열거 타입을 키로 사용하도록 설계한 FAST Map 구현체

```java
public static void usingEnumMap(List<Plant> garden) {
        Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);

        for (LifeCycle lifeCycle : LifeCycle.values()) {
            plantsByLifeCycle.put(lifeCycle,new HashSet<>());
        }

        for (Plant plant : garden) {
            plantsByLifeCycle.get(plant.lifeCycle).add(plant);
        }

        //EnumMap은 toString을 재정의하였다.
        System.out.println(plantsByLifeCycle);
    }
```
- 안전하지 않은 형변환 사용 X
- 배열 인덱스 계산 과정 오류 가능성 X
- 내부에서 배열을 사용해서 ordinal 배열만큼 성능 come out : 낭비 되는 공간과 시간이 거의 없어 명확하고 안전하고 유지보수 GOOD
___________________________________--
## ordinal 사용 안하고 enummap 사용해야 겠다 확실히 체감한 예시 

- 다음은 두 가지 상태(Phase)를 전이(Transition)와 맵핑하는 예제이다. LIQUID에서 SOLID의 전이는 FREEZE가 되고, LIQUID에서 GAS로의 전이는 BOIL이 될것이다.

`ORDINAL` 

```java 
public enum Phase {
// 0 1 2
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
        // [0][1] 일케 ORDINAL 기준으로 이차원 배열 값 부르는 것이지 
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```
- Phase나 Transition의 상수의 선언 순서를 변경하거나 새로운 Phase 상수를 추가하는 경우에도 문제가 발생
- 새로운 상태인 플라즈마(PLASMA)를 추가해보자. 플라즈마와 관련된 전이는 2개로 첫 번째는 기체에서 플라즈마로 변하는 이온화(IONIZE) 이고 두 번째는 반대인 탈이온화(DEIONIZE) 이다.
- 배열버전에선 이 상태들을 추가하는데 비용이 많이든다. 
   - 가령 원소 9개의 배열을 원소 16개의 배열로 교체해야하고, 만일 잘못된 순서로 나열하면 런타임 오류가 발생

`ENUM MAP`
```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> transitionMap;

        static {
            transitionMap = Stream.of(values())
                    .collect(Collectors.groupingBy(t -> t.from, // 바깥 Map의 Key
                            () -> new EnumMap<>(Phase.class), // 바깥 Map의 구현체
                            Collectors.toMap(t -> t.to, // 바깥 Map의 Value(Map으로), 안쪽 Map의 Key
                                    t -> t, // 안쪽 Map의 Value
                                    (x,y) -> y, // 만약 Key값이 같은게 있으면 기존것을 사용할지 새로운 것을 사용할지
                                    () -> new EnumMap<>(Phase.class)))); // 안쪽 Map의 구현체
        }

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }
}
```

```

1. 일단 from으로 grouping 하여 EnumMap을 하나 생성
2. toMap으로 하위 Map을 생성
3. 첫 번째 인자는 Map의 key를 설정하는 Function  - Phase to로 선언
4. 두 번째 인자는 Map의 value를 설정하는 Function  - 자기 자신을 참조
5. 세 번째 인자는 merge-function이다. - 얘는 별 의미 X
6. 네 번째 인자는 EnumMap으로 내부 Map을 선언 

```

- Phase에 PLASMA를 추가하고 TRANSITION에 두 전이 상태를 추가하면 끝
```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);
    }
}
```

## REFERENCE 
https://jyami.tistory.com/105?category=908686
https://javabom.tistory.com/51