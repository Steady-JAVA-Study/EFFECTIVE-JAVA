# ITEM 26. 로 타입(raw type)은 사용하지 말라

클래스와 인터페이스 선언에 타입 매개변수(Type Parameter)가 사용되면, 이를 제네릭 클래스 혹은 제네릭 인터페이스라고 부르며 이를 통틀어 `제네릭 타입`이라고 한다.

> 로 타입?
> 
> 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 것을 의미 -> List<E>의 로 타입은 List 이다.

## 로 타입을 사용하면 안되는 이유

로 타입의 가장 큰 문제점은 컴파일 시점에 에러를 잡을 수가 없다는 것이다.

실제로 값을 가져와서 캐스팅을 하는 시점에 오류가 발생하므로, 컴파일러는 ClassCastException을 던진다.

```java
List onlyString = new ArrayList();
onlyString.add("문자열1");
onlyString.add("문자열2");
onlyString.add(13579);

for (Iterator i = onlyString.iterator(); i.hasNext(); ) {
  String s = (String) i.next(); // ClassCastException 을 던진다.
}
```

**List<Object>** 는 명확하게 Object라는 타입을 제시한 것이므로 오해하지 말자.

## 제네릭을 사욯하자!

제네릭을 사용하면 컴파일 시점에 에러를 발생시킨다.

```java
List<String> strings = new ArrayList<String>();
strings.add("string1");
strings.add("string2");
strings.add(13579); // error: incompatible types: int cannot be converted to String
```

## 제네릭은 런타임에 무조건 안전할까?

컬렉션이 파라미터화되어 선언되었더라도, 이 컬렉션을 인자로 받는 메서드의 파라미터가 로 타입이라면 런타임 오류가 발생할 수 있다.

```java
public class GenericsRuntimeError {
	public static void main(String[] args) {
		List<String> strings = new ArrayList<>();
		unsafeAdd(strings, Integer.valueOf(42));
		String s = strings.get(0); // 런타임 에러, class java.lang.Integer cannot be cast to class java.lang.String
	}

	private static void unsafeAdd(List list, Object o) {
		list.add(o);
	}
}
```

## 컬렉션의 원소 타입을 모를 때 혹은 컬렉션에 어떤 원소 타입이든 넣고 사용하려면?

컬렉션의 타입 파라미터를 모르거나 신경쓰고 싶지 않다면, **비한정적 와일드 카드 타입(unbounded wildcard type)**을 사용하자.

예를 들어, 다음과 같이 Set<E>에서 E가 들어갈 자리에 **?**를 사용하면 된다.

로 타입으로 사용했을 때와, 비한정적 와일드 카드 타입으로 사용할 때 차이점은 무엇일까?

```java
public class UnboundedWildCard {
	public static void main(String[] args) {
		Set rawTypeSet = new HashSet();
		Set<?> wildcardSet = new HashSet<>();

		rawTypeSet.add("string1");
		rawTypeSet.add(Integer.valueOf(13579));

		wildcardSet.add("string1"); // 컴파일 에러 발생

	}
}
```

와일드 카드를 지정한 wildcardSet 컬렉션에 문자열 타입을 넣거나 아무 타입을 추가해보면(null을 제외한), 다음과 같은 컴파일 에러가 발생한다.

> error: incompatible types: String cannot be converted to CAP#1 wildcardSet.add("string1"); ^ where CAP#1 is a fresh type-variable: CAP#1 extends Object from capture of ?

### 와일드 카드는 언제 사용할까?

다음과 같이 메서드 파라미터에 사용할 수 있다. 이 메서드는 Set 컬렉션이 어떠한 타입이든지 인자로 받을 수 있으므로, 특정 파라미터에 맞게 일일이 메서드를 정의할 필요가 없다.

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
  int result = 0;
  for (Object o1 : s1) {
    if (s1.contains(s2))
      result++;
  }
  return result;
}
```