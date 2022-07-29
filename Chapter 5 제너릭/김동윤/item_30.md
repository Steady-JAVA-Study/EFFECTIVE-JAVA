# 이왕이면 제네릭 메서드로 만들라
- 클래스뿐만 아니라 메서드도 제네릭화 possible 
- 대표적인 예로 매개변수를 받는 정적 유틸리티 메서드(Collections의 알고리즘 메서드 binarySearch, sort ...)이 보통 제네릭 메서드

```java
public static Set union(Set s1, Set s2) {
	Set result = new HashSet(s1);
	result.addAll(s2);
	return result;
}
```
> warning: [unchecked] unchecked

-  메서드 선언에서 Set의 원소 타입을 타입 매개변수로 명시 & 메서드 안에서도 타입 매개변수만 사용하게 수정

```java

public static Set<E> union(Set<E> s1, Set<E> s2) {

    Set<E> result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

## 결론
- 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가  better
- 형변환은 좋지 않으니 제네릭 메서드를 사용할 수 있다면 무조건 제네릭 메서드 used ! 