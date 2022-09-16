# ITEM 86. Serializable 을 구현할지는 신중히 결정하라

## Warning Point

1. Serializable을 구현하면 릴리스한 뒤에는 수정하기 어렵다.

클래스가 Serializable을 구현하게 되면 직렬화된 바이트 스트림 인코딩도 하나의 공개 API가 된다. 때문에 이 클래스가 널리 퍼지면 그 직렬화 형태도 영원히 지원해야한다. 즉, Serializable을 구현한 순간부터 해당 객체의 직렬화 형태는 Java 직렬화에 묶이는 것이다. 기본 직렬화 형태에서는 private와 package-private 수준의 필드마저도 API로 공개가 된다. 즉, 캡슐화가 깨진다.

2. 버그와 보안 구멍이 생길 위험이 높아진다.

객체를 생성하는 가장 기본적인 방법은 생성자를 이용하는 것이다. 근데 ObjectInputStream#readObject는 객체를 만들어 낼 수 있는 마법같은 메서드이다. 즉, 객체를 Serializable로 구현하면 생성자 이외에 객체를 생성할 수 있는 숨은 생성자가 생기는 것이다.

3. 테스트 범위가 늘어난다.

4. 상속용으로 설계된 클래스는 Serializable을 구현하면 안되며, 인터페이스도 Serializable을 확장하면 안된다.
5. 내부 클래스는 Serializable을 구현하면 안된다.

