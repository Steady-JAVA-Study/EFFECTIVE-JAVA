# ITEM 7. 다 쓴 객체 참조를 해제하라

자바는 가비지 컬렉터(GC)가 메모리를 관리해준다.

GC가 있다고 해서 메모리 관리를 개발자가 신경을 쓰지 않아도 되는 것이 아니다.

```java
public class Stack {
	private Obejct[] elements;
	private int size = 0;
    private static final int DEFAULT_INITAL_CAPACITY = 16;

    public Stack() {
		elements = new Object[DEFAULT_INITAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++];
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		return elements[--size];
	}

	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
}
```

위의 스택은 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다

`elements[--size]` 이와 같이 반환값을 해놓는다면 실제 값은 삭제되지 않고 인덱스만 한칸씩 이동하는 것으로 메모리 누수가 발생하게 된다.

### 해결방안

해법은 간단하다. 해당 참조를 다 썼을 때 null 처리(참조 해제)하면 된다.
```java
public Object pop() {
	if (size == 0) {
		return new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null;
	return result;
```

null을 사용함으로써 또 다른 이점은 만약 null처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 NPE(NullPointerException)을 던지며 종료한다.

### 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.

참조 해제의 가장 좋은 방법은 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.

이 부분은 `아이템 57. 지역변수의 범위를 최소화하라`를 참고하면 알 수 있다.

스택 클래스는 자기 메모리를 직접 관리학 때문에 메모리 누수에 취약하다. 위의 스택은  elements 배열로 저장소 풀을 만들어 원소들을 관리한다. 문제는 가비지 컬렉터가 이 사실을 알 수 없다.

가비지 컬렉터가 보기에는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체다. 비활성 영역의 객체가 더 이상 쓸모가없다는 건 프로그래머만 아는 사실이다. 그러므로 프로그래머는 비활성 영역이 되는 순간 null처리해서 해당 객체를 더는 쓰지 않을 것임을 가비지 컬렉터에 알려야 한다.

**일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.**

### 캐시 역시 메모리 누수를 일으키는 주범이다.

객체 참조를 캐시에 넣고, 객체를 다 쓴 뒤에도 한참을 놔두는 일이 자주 있다.

```java
Object key1 = new Object();
Object value1 = new Object();

Map<Object, List> cache = new HashMap<>();
cache.put(key1, value1);
```

key1이 없어지면 이 캐싱 자체가 무의미해지는 경우가 많다.

### 해결방안 - `WeakHashMap`

캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 `WeakHashMap`을 사용해 캐시를 만들자. 다 쓴 엔트리는 자동으로 제거된다.
```java
Object key1 = new Object();
Object value1 = new Object();

Map<Object, List> cache = new WeakHashMap<>();
cache.put(key1, value1);
```

### 리스너(listener) 혹은 콜백(callback)은 메모리 누수의 주범이다.

클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다. 이럴 때 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해간다. 

예를 들어 `WeakHashMap`에 키로 저장하면 된다.

> 강한 참조(Strong Reference) ? 부드로운 참조(Soft Reference) ? 약한 참조(Weak Reference) ?
> 
> 강한 참조: 일반적으로 new를 통해서 객체를 생성하게 되면 생기게 되는 참조
> 
> 강한 참조를 통해 참조되고 있는 객체는 가비지 컬렉션의 대상에서 제외된다.
> 
> 부드러운 참조: 강한 참조와는 다르게 GC에 의해 수거될 수도 있고, 수거되지 않을 수도 있다. 메모리에 충분한 여유가 있다면 GC가 수행되고 잇다 하더라도 수거되지 않는다. 하지만 out of memory의 시점에 가깝다면 수거될 확률이 높다.
>
> 약한 참조: 약한 참조는 GC가 발생하면 무조건 수거된다. WeakReference가 사라지는 시점이 GC의 실행 주기와 일치하며 이를 이용하여 짧은 주기에 자주 사용되는 객체를 캐시할 때 유용하다.
>
> https://ktko.tistory.com/entry/%EC%9E%90%EB%B0%94-%EA%B0%95%ED%95%9C%EC%B0%B8%EC%A1%B0Strong-Reference%EC%99%80-%EC%95%BD%ED%95%9C%EC%B0%B8%EC%A1%B0Weak-Reference
---

## WeakHashMap

```java
public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.gc();  //강제 Garbage Collection

        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
```

```java
2000=test b
1000=test a
```

```java
public static void main(String[] args) {
        Map<Integer, String> map = new WeakHashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.gc();  //강제 Garbage Collection

        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
```

```java
2000=test b
```