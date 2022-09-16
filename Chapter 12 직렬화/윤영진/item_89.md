# ITEM 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

싱글톤을 유지하고 싶은 클래스에 Serialize을 구현하면 해당 클래스는 싱글톤이 아니게된다.

```java

public class item89 {
	public static void main(String[] args) throws IOException, ClassNotFoundException {
		Elvis elvis = Elvis.getInstance();

		byte[] serialized;
		// 직렬화
		try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
			try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
				oos.writeObject(elvis);
				serialized = baos.toByteArray();
			}
		}

		// 역직렬
		Elvis deserializedElvis;
		try (ByteArrayInputStream bais = new ByteArrayInputStream(serialized)) {
			try (ObjectInputStream ois = new ObjectInputStream(bais)) {
				 deserializedElvis = (Elvis) ois.readObject();
			}
		}
    
		System.out.println(elvis == deserializedElvis);
		System.out.println(format("before: %s, after: %s", elvis, deserializedElvis));
	}
}
```

`readObject` 메서드로 인해 인스턴스가 생성된다. 

만약 `Serializable`을 반드시 구현해야하며, 그 인스턴스는 반드시 싱글톤이어야만 한다면 `readResolve`를 사용해야 한다.

readResolve() 메서드는 역직렬화 후에 인스턴스를 생성하기 위해 호출된다.

```java
public class Elvis implements Serializable {

	private static final Elvis INSTANCE = new Elvis();

	private Elvis() {}

	public static Elvis getInstance() { return INSTANCE; }

	private Object readResolve() {
		return INSTANCE;
	}
}
```

단, `readResolve` 메서드는 역직렬화 시점에 필드가 nontransient 래퍼런스 타입이라면 다른 객체로 바뀔 위험이 있다.

이러한 공격을 방지하기 위해 래퍼런스 필드는 `transient` 로 선언해야 한다.

하지만 더 간결한 방법으로 해당 클래스를 enum 타입으로 만드는 것이다.

Enum 타입과 같이 인스턴스가 통제된 직렬화 가능한 클래스를 작성하면 선언된 객체 이외에는 어떠한 인스턴스도 있지 않음을 보장한다.

```java
public enum Elvis {
  INSTANCE;
  private String[] referenceField = {"One","Two", "Tre"};
  public String[] getReferenceField() { ... }
}
```

---

Deserialize 중에 readObject 메서드가 존재하더라도 readResolve 메서드에서 반환한 인스턴스로 대체된다. readObject를 통해 만들어진 건 바로 객체 참조가 사라진다.

readObject도 수행되고 readResolve 또한 수행되는 것이다.