# ITEM 30. 이왕이면 제네릭 메서드로 만들어라

## 제너릭 메서드의 정의

```java
public static <T> Box<T> getBox(T o) { ...}
```

제네릭 타입과 같이 형변환 해야하는 메서드보다 제네릭 메서드가 더 안전하고, 심지어 사용하기도 쉽다.

형변환이 필요한 메서드는 제너릭으로 만들자.

## 제너릭없는 메서드 

```java
public static Set union(Set s1, Set s2) {
	Set result = new HashSet(s1);
	result.addAll(s2);
	return result;
}
```

위의 메서드는 타입이 안전하지 않다는 경고를 낸다.

## 제너릭 메서드

```java
public static <E> set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```


