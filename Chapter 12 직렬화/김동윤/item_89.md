# 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

## readResolve?
> readResolve 메서드를 이용하면 readObject 메서드가 만든 인스턴스를 다른 것으로 대체할 수 있다. 이때 readObject 가 만들어낸 인스턴스는 가비지 컬렉션의 대상이 된다.

- 자바 직렬화에 사용되는 readResolve 메서드
- 클래스의 객체 개수(보통 1개)를 제어하는 방법을 싱글톤 패턴
- 객체가 여러 개 생성될 필요가 없을 때 하나만 생성하여 사용할 때마다 같은 객체를 참조
- 하지만 싱글톤 클래스는 직렬화 가능한 클래스가 되기 위해 Serializable 인터페이스를 구현(implements) 하는 순간, 싱글톤 클래스가 아닌 상태가 된다. 직렬화를 통해 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환
=> 이럴 때 readResolve 메서드를 사용하면 된다. 이 메서드를 직접 정의하여 역직렬화 과정에서 만들어진 인스턴스 대신에 기존에 생성된 싱글톤 인스턴스를 반환
- 만일 역직렬화 과정에서 자동으로 호출되는 readObject 메서드가 있더라도 readResolve 메서드에서 반환한 인스턴스로 대체
- readObject 메서드를 통해 만들어진 인스턴스는 가비지 컬렉션 대상

## 그래도 readResolve보다는 해결책은 enum
-  readResolve 메서드를 인스턴스의 통제 목적으로 이용한다면 모든 필드는 transient로 선언해야 함 ;;
- 만일 그렇지 않으면 역직렬화(Deserialization) 과정에서 역직렬화된 인스턴스를 가져올 수 있음 => 싱글턴 깨져
- 열거 타입으로 싱글턴을 구현한다면 선언한 상수외 다른 객체가 존재하지 않음을 보장
   -  enum을 사용하면 모든 것이 해결된다. 자바가 선언한 상수 외에 다른 객체가 없음을 보장
## Nevertheless,,
- readResolve() 메서드를 사용하는 방식이 아예 필요없는 것은 아니다.
열거타입으로 표현이 불가능할 때 사용할 수 있다.
해당 메서드의 접근성은 중요하다.
- final 클래스라면 readResolve() 메서드는 private이어야 한다.
protected나 public으로 선언한다면 재정의하지 않은 모든 하위클래스에서 사용할 수 있는데, 이 때 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException이 발생할 수 있다

### reference
https://greeng00se.tistory.com/115
https://madplay.github.io/post/for-instance-control-prefer-enum-types-to-readresolve