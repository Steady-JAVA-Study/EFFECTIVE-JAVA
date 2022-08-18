# ITEM 45. 스트림은 주의해서 사용하라

## Stream의 구조

```java
user.stream() // 스트림 생성
        .filter(x -> x.name == 'yoon') // 중간 연산
        .count(); // 종단 연산
```


### 스트림(Stream)

- **stream** : 객체 참조에 대한 Stream
- **intStream** : int 타입에 대한 Stream
- **longStream** : Long 타입에 대한 Stream
- **doubleStream** : Double 타입에 대한 Stream

### 중간 연산(Intermediate operation)

-   **filter(Predicate<? super T> predicate)**  : predicate 함수에 맞는 요소만 사용하도록 필터

-   **map(Function<? Super T, ? extends R> function)**  : 요소 각각의 function 적용

-   **flatMap(Function<? Super T, ? extends R> function)**  : 스트림을 하나의 스트림으로 변경

-   **distinct()**  : 중복 제거

-   **sort()**  : 정렬

-   **sort(Comparator<? super T> comparator)**  : comparator 함수를 이용하여 정렬

-   **skip(long n)**  : n개 만큼의 스트림 요소 건너뜀

-   **limit(long maxSize)**  : maxSize 갯수만큼만 출력

### 종단 연산(Terminal operation)

- **forEach(Consumer<? super T> consumer)**  : Stream의 요소를 순회

- **count()**  : 스트림 내의 요소 수 반환

- **max(Comparator<? super T> comparator)**  : 스트림 내의 최대 값 반환

- **min(Comparator<? super T> comparator)**  : 스트림 내의 최소 값 반환

- **allMatch(Predicate<? super T> predicate)**  : 스트림 내에 모든 요소가 predicate 함수에 만족할 경우 true

- **anyMatch(Predicate<? super T> predicate)**  : 스트림 내에 하나의 요소라도 predicate 함수에 만족할 경우 true

- **noneMatch(Predicate<? super T> predicate)**  : 스트림 내에 모든 요소가 predicate 함수에 만족하지않는 경우 true

- **sum()**  : 스트림 내의 요소의 합 (IntStream, LongStream, DoubleStream)

- **average()**  : 스트림 내의 요소의 평균 (IntStream, LongStream, DoubleStream) 

- **collect()** :  스트림 인터페이스에 대해서 list나 set으로 변경 

## 종단 연산(terminate operation)을 빼먹는 일이 절대 없도록 하자.

스트림에 대한 연산은 종단 연산 끝에 일어나기 때문에 항상 스트림을 사용하면, 종단 연산으로 끝을 내줘야 한다.

## 지연 평가(lazy evaluation)의 개념에 대해서 알자.

호출되는 순간에 해당 코드에서 사용하는 변수와 문장이 실행되도록 하는 것을 Eager code라고 한다.

하지만, 실행할 때 몇몇 코드를 실행 순서가 됐을 때 바로 실행하기 보다는 약간의 lazy 시키는 것이 성능 향상의 방법이 될 수 있다.

그 중 Lambda에서 쓰는 방법이 바로 lazy evaluation이다.

filter().map().findFirst() 를 통하여 특정 데이터를 가지고 온다면,

Eager code인 경우 차례대로 실행하기 때문에 많은 시간이 소모되게 된다.

하지만, Lambda가 관여하면, filter(), map() 메서드는 본질적으로 레이지 속성을 가지기 때문에, filter(), map() 메서드는 람다 표현식을 저장하고, 체인의 다음 호출에게 이 람다 표현식을 전달한 다음에, 마지막에 종단 연산이 호출 할 때 사용하게 된다.

```java
public static void main(String[] args) {  
    List<String> names = Arrays.asList("Brad", "Kate", "Kim",  "Jack", "Joe", "Mike", "Susan", "George", "Robert", "Julia", "Parker", "Benson");  
    final Stream<String> firstNameWith3Letters = names.stream().filter(name -> name.length() == 3);  
    
    System.out.println("names.size() = " + names.size()); 
     
    Stream<String> stringStream= firstNameWith3Letters.map(String::toUpperCase);  
    
    System.out.println("names.size() = " + names.size()); 
     
    String first = stringStream.findFirst().get();  
    
    System.out.println("names.size() = " + first);  
}
names.size() = 12
names.size() = 12
names.size() = KIM
```

### 과도한 스트림의 사용은 피하라.


스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.

### char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.

```java
"Hello Char".char().forEach(Systeom.out::print);
```

해당 부분에서 출력을 하면, 정수값들을 출겨하게 된다.

그러므로 아래와 같이 형변환을 명시적으로 해줘야 한다.

```java
"Hello Char".char().forEach(x -> System.out.printlin((char) x));
```

### 스트림 안에서는 final 변수만 읽을 수 있다.

Stream 안에서는 final 변수 / effectively final 변수만 사용이 가능하다.

> effectively final 이란? 
> 
> final이 붙지 않았으나, 변수의 값이 변경되지 않은 변수

스트림 안에 변수가 동시성 이슈를 대응할 수 없기 때문에, 스트림 안에서는 final 변수 만 읽을 수 있다.

### 람다로는 코드 블록에서의 return, break, continue, 예외를 던질 수 없다.


--- 

# Advanced Stream

## Collectors

스트림 데이터를 어떻게 그룹화 하냐에 따라 여러 방식이 나뉜다.

- 요약

  - counting
  - maxBy, minBy
  - summingInt, summingLong, summingDouble
  - averagingInt, averagingLong, averagingDoubl
  - summarizingInt, summarizingLong, summarizingDouble
  - joining
  - toList, toSet, toCollection
  
- 다수준 그룹화
  - groupingBy, collectingAndThen

- 분할
  - partitioningBy

## 요약

```java
List<User> users = new ArrayList<>(List.of(
                new User(1, "yoon1"),
                new User(2, "yoon2"),
                new User(3, "yoon3")
        ));
```

### 개수 모으기 - counting
```java
Long count = users.stream().collect(Collectors.counting()); // 3
// Long count = users.stream().count();

```

### 최댓값과 최솟값 - maxBy, minBy

```java
Comparator<User> userIdxComparator = Comparator.comparingInt(User::getIdx);
        Optional<User> longestIdxUser = users.stream().collect(Collectors.maxBy(userIdxComparator));
        //        Optional<User> longestIdxUser = users.stream().max(userIdxComparator);

        Optional<User> shortestIdxUSer = users.stream().collect(Collectors.minBy(userIdxComparator));
        //         Optional<User> shortestIdxUSer = users.stream().min(userIdxComparator);
        System.out.println(longestIdxUser.get().getName()); // yoon3
        System.out.println(shortestIdxUSer.get().getName()); // yoon1

```

### 숫자 타입 요약(합계, 평균, 통계)

```java
Integer collect = users.stream().collect(Collectors.summingInt(User::getIdx));
//         Integer collect = users.stream().mapToInt(User::getIdx).sum();
System.out.println(collect); // 6

Double avg = users.stream().collect(Collectors.averagingDouble(User::getIdx));
System.out.println(avg);  // 2.0


```

### 문자열 연결

```java
String joining = users.stream().map(User::getName).collect(Collectors.joining(", "));
System.out.println(joining); // yoon1, yoon2, yoon3 
```

### 기본에 충실한 요약 - reducing

기본적으로 파라미터를 받는데, (초기값[또는 스트림이 비었을 때 값], 변환 함수, 같은 종류의 두 항목을 하나로 만드는 함수) 이렇게 3개다.

```java
Integer collect = users.stream().collect(Collectors.reducing(0, User::getIdx, (x, y) -> x + y));
// Integer collect = users.stream().map(User::getIdx).reduce(0, (x, y) -> x + y);
System.out.println(collect); // 6
```

## 그룹화

```java
List<User> users = new ArrayList<>(List.of(
                new User(1, "yoon1", User.Position.MANAGER),
                new User(2, "yoon2", User.Position.MANAGER),
                new User(3, "yoon3", User.Position.CEO)
        ));
```

### groupingBy


```java
Map<User.Position, List<User>> usersByPosition = users.stream().collect(groupingBy(User::getPosition));
// {MANAGER=[User{idx=1, name='yoon1', position=MANAGER}, User{idx=2, name='yoon2', position=MANAGER}], CEO=[User{idx=3, name='yoon3', position=CEO}]}

```

#### 요구사항이 존재하는 groupingBy
```java
Map<User.Position, List<User>> filteredUserByPosition = users.stream().collect(Collectors.groupingBy(User::getPosition,
                        Collectors.filtering(user -> user.getIdx() > 2, Collectors.toList())));
// {CEO=[User{idx=3, name='yoon3', position=CEO}], MANAGER=[]}
```

```java
Map<User.Position, List<Integer>> mappedUserByPosition = users.stream().collect(Collectors.groupingBy(User::getPosition,
                Collectors.mapping(User::getIdx, Collectors.toList())));

// {CEO=[3], MANAGER=[1, 2]}
```

#### 다수준 그룹화

```java
  Map<User.Position, Map<User.Department, List<User>>> usersByPositionDepartment =
                users.stream().collect(
                        Collectors.groupingBy(User::getPosition,
                                Collectors.groupingBy(User::getDepartment))
                );

        System.out.println(usersByPositionDepartment);
// {MANAGER={D1=[User{idx=1, name='yoon1', position=MANAGER}, User{idx=2, name='yoon2', position=MANAGER}]}, STAFF={D1=[User{idx=4, name='yoon4', position=STAFF}]}, CEO={D2=[User{idx=3, name='yoon3', position=CEO}]}}
```

## 분할

분할은 그룹화랑 다른 게 하나 밖에 없다.

그룹화는 N개의 그룹으로 그룹화할 수 있는 반면, 분할은 무조건 2개의 그룹으로 그룹화할 수 있다.

ex. (비정규직, 정규직)

```java
Map<Boolean, List<User>> contractedEmployee = users.stream().collect(Collectors.partitioningBy(User::isFulltimeJob));
// {false=[User{idx=3, name='yoon3', position=CEO}], true=[User{idx=1, name='yoon1', position=MANAGER}, User{idx=2, name='yoon2', position=MANAGER}, User{idx=4, name='yoon4', position=STAFF}]}
```

