# ITEM 33. 타입 안전 이종 컨테이너를 고려하라

> 타입 안전 이종 컨테이너(type safe heterogeneous container)?
> 
> **여러 다른 종류**들로 이루어진 **값을 저장하는 타입에 안전한 객체**를 뜻한다.

## 타입 이종 컨테이너를 사용하는 이유?

학번을 저장하는 Set<Integer> -> Integer형의 값만 저장할 수 있다.

이름과 나이를 저장하는 Map<String, Integer> -> String형의 key와 Integer형의 value만 저장할 수 있다.

하지만 하나의 Map에 Integer뿐 아니라 String, Double, Boolean 타입도 담고 싶을 경우에는? **타입 안전 이종 컨테이너**

## 사용법

방법은 간단하다. 바로 컨테이너를 매개변수화 하지 않고 컨테이너의 키를 매개변수화 하면 된다.

Map<String, Integer> : 컨테이너를 매개변수화 했다.

Map<Class<?>, Object> : 컨테이너의 키를 매개변수화 했다.

다음은 타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스이다.

```java
private Map<Class<?>, Object> favorites = new HashMap<>();

public <T> void putFavorites(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
}

public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
}
```

```java
@Test
  public void 타입_안전_이종_컨테이너_패턴() {
    // given
    Favorites f = new Favorites();
    
    String testString = "Effective Java";
    Integer testInteger = 1;
    Class testClass = Favorites.class;
    

    // when
    f.putFavorite(String.class, testString);
    f.putFavorite(Integer.class, testInteger);
    f.putFavorite(Class.class, testClass);
    
    // then
    Assert.assertEquals(f.getFavorite(String.class), testString);
    Assert.assertEquals(f.getFavorite(Integer.class), testInteger);
    Assert.assertEquals(f.getFavorite(Class.class), testClass);
  }
```

