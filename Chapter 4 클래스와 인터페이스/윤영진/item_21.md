# ITEM 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8 이후 interface에 default 메서드가 추가되었다.

하지만, 아직까지는 default method를 작성하는 것은 어렵다.

## Collection 인터페이스의 default 메서드인 removeIf 메서드

```java
default boolean removeIf(Predicate<? super E> filter) {  
	Objects.requireNonNull(filter);  
	boolean removed = false;  
	final Iterator<E> each = iterator();  
	while (each.hasNext()) {  
		 if (filter.test(each.next())) {  
			 each.remove();  
			 removed = true;  
		 }  
	}  
	return removed;  
}
```

```java
List<String> list = new ArrayList<>();  
list.add("1");  
list.add("2");  
list.add("1");  
list.add("2");  
list.add("3");  
  
System.out.println("list.size() = " + list.size());   // 5
  
list.removeIf(number -> number.equals("1"));  
  
System.out.println("list.size() = " + list.size()); // 3
```

Collection의 RemoveIf는 현재 범용적으로 사용을 하고 있지만, 아직까지 모든 Collection 구현체와 잘 어울리는 게 아니라고 한다.

책에서는 SynchronizedCollection을 예를 들었고, 해당 SynchroizedCollection을 사용했을 때 RemoveIf를 사용하게 되면 ConcurrentModificationException을 발생시키거나, 다른 결과를 보내준다고 한다.


