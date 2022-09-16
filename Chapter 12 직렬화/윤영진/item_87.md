# ITEM 87. 커스텀 직렬화 형태를 고려하라

```java
public class Item87 {
    static class Person implements Serializable {
        private Long id;
        private String name;
        private float height;
        private String trashData;

        public byte[] serialize() {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            try (bos; ObjectOutputStream oos = new ObjectOutputStream(bos)) {
                oos.writeObject(this);
            } catch (IOException e) {
            }
            return bos.toByteArray();
        }

        public static Person deserialize(byte[] data) {
            ByteArrayInputStream bis = new ByteArrayInputStream(data);
            try (bis; ObjectInputStream ois = new ObjectInputStream(bis)) {
                return (Person) ois.readObject();
            } catch (Exception e) {

            }
            return null;
        }
    }

    
}
```

```java
public static void main(String[] args) {
        Person p1 = new Person(1L, "홍길동", 160.4f, "Trash");
        System.out.println(p1);
        byte[] serializeData = p1.serialize();
        Person p2 = Person.deserialize(serializeData);
        System.out.println(p2);
    }
```

## Custom Serialize (writeObject & readObject)

```java
private void readObject(ObjectInputStream ois) {
            try {
                System.out.println("Test");
                ois.defaultReadObject();
                this.id = (Long) ois.readObject();
                this.trashData = (String) ois.readObject();
                this.height = (Float) ois.readObject();
                this.name = (String) ois.readObject();
            }catch (IOException | ClassNotFoundException e) {

            }
        }

        private void writeObject(ObjectOutputStream oos) {
            try {
                System.out.println("Test2");
                oos.defaultWriteObject();
                oos.writeObject(this.id);
                oos.writeObject(this.name);
                oos.writeObject(this.height);
                oos.writeObject(this.trashData);

            }catch (IOException e) {

            }
        }
```

위와 같이 커스텀함으로써 같은 타입의 id와 name이 순서에 맞지 않게 입력되어도 괜찮아진다.

---

## transient

Trasient로 선언한 필드는 직렬화 대상에서 제외된다.

## UID

직렬화 가능한 클래스에 UID를 명시해 두면, 직렬 버전 UID가 일으키는 잠재적 호환성 문제가 사라진다. 또한 런타임시 이 값을 생성하지 않아도 되기 때문에 속도 향상도 있다.

대신 UID가 수정되면 호환성이 깨진다.
