# ITEM 88. readObject 메서드는 방어적으로 사용하라

readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다. 

객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 방어적으로 복사해야 한다.

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
   s.defaultReadObject();

   // 방어적 복사를 통해 인스턴스의 필드값 초기화
   start = new Date(start.getTime());
   end = new Date(end.getTime());

   // 유효성 검사
   if (start.compareTo(end) > 0)
       throw new InvalidObjectException(start +" after "+ end);
}
```

## 안전한 readObject

1. Private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라.
2. 모든 불변식을 검사하여 어긋나는 것이 발견되면 `InvalidObjectException`을 던진다.
3. Desrialize 후 Object Graph의 유효성을 검사해야 한다면 `ObjectInputValidation Interface` 사용
4. 직/간접적으로 재정의할 수 있는 메서드는 호출하지 말자