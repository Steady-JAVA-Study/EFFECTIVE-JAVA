# ITEM 64. 객체는 인터페이스 타입으로 참조하라

## 객체는 클래스가 아닌 인터페이스 타입으로 선언하라

```java
//좋은 예 
Set<String> set = new LinkedHashSet<>();

//나쁜 예
LinkedHashSet<String> set = new LinkedHashSet<>();
```

인터페이스를 타입으로 사용하면 프로그램이 훨씬 유연해진다.

나중에 구현 클래스를 교체할 때 새 클래스의 생성자를 호출해주기만 하면 된다.

```java
Set<Son> sonSet = new HashSet<>();
```

## 적합한 Interface가 없는 경우에는 클래스로 참조하라

예를 들어 인터페이스에는 없는 특별한 메서드를 제공하는 클래스들이다.
```java
List<Integer> list = new ArrayList<>();
ArrayList<Integer> list2 = new ArrayList<>();

// list.ensureCapacity
list2.ensureCapacity(3);
```

ensureCapacity() 같은 메서드는 ArrayList에만 구현되어 있다. 그래서 List 인터페이스로 받으면 사용할 수 없다.

적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인 클래스를 타입으로 사용하는 것을 추천한다.