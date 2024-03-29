# ITEM 82. 스레드 안전성 수준을 문서화하라

## 스레드 안전성이 높은 순

1. 불변(Immutable)
- 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화도 필요 없다.
- String, Long, BigInteger가 대표적이다.
2. 무조건적 스레드 안전(Unconditionally thread-safe)
- 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다.
- AtomoicLong, ConcurrentHashMap이 여기에 속한다.
3. 조건부 스레드 안전(conditionally thread-safe)
- 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다.
- Collections.synchronized 래퍼 메서드가 반환한 컬렉션들이 여기 속한다(이 컬렉션들이 반환한 반복자는 외부에서 동기화해야 한다).
4. 스레드 안전하지 않음(not thread-safe)
- 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의(혹은 일련의) 메서드 호출을 클라이언트가 선택한 외부 초기화 메커니즘으로 감싸야 한다.
- ArrayList, HashMap같은 기본 컬렉션이 여기 속한다.
5. 스레드 적대적(thread-hostile)
- 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다. 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정한다. 이런 클래스를 고의로 만드는 사람은 없겠지만, 동시성을 고려하지 않고 작성하다 보면 우연히 만들어질 수 있다.

