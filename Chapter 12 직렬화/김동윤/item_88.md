# readObject 메서드는 방어적으로 작성하라

## readObject?
- 기본적인 자바 직렬화 또는 역직렬화 과정에서 별도의 처리가 필요할 때는 writeObject와 readObject 메서드를 클래스 내부에 선언
- 바이트 스트림을 인수로 받는 일종의 public 생성자
- 인수가 유효한지 검사해야하고 필요하다면 매개변수를 방어적으로 복사
- 객체를 역직렬화 할때는 클러아인트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사

## 결론
- 안전한 readObject 메서드를 작성하는 지침
- private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기 속함
- 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 함
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용 
- 직접적이든, 간접적이든, 재정의할 수 있는 메서드는 호출 X
### reference
https://madplay.github.io/post/what-is-readobject-method-and-writeobject-method

https://velog.io/@wlghsp/Item-88-readObject-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EB%B0%A9%EC%96%B4%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%9E%91%EC%84%B1%ED%95%98%EB%9D%BC