# 상속보다는 컴포지션 사용해라
## 상속은 캡슐화를 깨뜨리지
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생기기 가능
- 상위 클래스의 내부 구현이 달라지면 하위 클래스를 고쳐야 할 수도 있음
- 다중상속이 불가능하다.
- 따라서 다른 클래스를 상속받고있다면 추가적으로 상속을 받을 수 없다.

## 컴포지션 (Composition)

>  서브클래스에 상위클래스로 하려고 했던 기존 클래스의 인스턴스를 참조하는 private필드를 참조하는 private 필드 추가 
[출처] Java - 상속(Inheritance)과 구성(Composition)|작성자 곰소

```java
/* 부모클래스 */
public class Notebook {

  public void on() {
    System.out.println("전원동작");
  }
}

/* 자식클래스 / 기존클래스 */
public class Macbook extends Notebook {

  private final Keyboard checker;

  /* 새로운 클래스를 기존클래스에 인스턴스 변수를 추가하여 사용 */
  public Macbook(Keyboard checker) {
    this.checker = checker;
  }

}

/* 컴포지션으로 사용할 클래스 */
public class Keyboard {

  public void keyStroke() {
    System.out.println("키입력");
  }
}
```
https://tgyun615.com/16

새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하는 방법
→ 기존 클래스가 새로운 클래스의 구성요소로 쓰이게 되는 구조

> - 다른객체의 인스턴스를 자신의 인스턴스 변수로 포함해서 메서드를 호출하는 기법이다.
- 해당 인스턴스의 내부 구현이 바뀌더라도 영향을 받지 않는다.
- 또한, 다른객체의 인스턴스이므로 인터페이스를 이용하면 Type을 바꿀 수 있다.

> - 새로운 클래스는 기존의 클래스의 구현방식을 벗어남
- 기존 클래스에 새로운 메소드가 추가되더라도 전혀 영향을 받지 않는다.
## 결과
- 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 사용
- 클래스 간에 'is-a' 관계 일때인 경우에만 상속을 사용

## 출처
https://iyoungman.github.io/designpattern/Inheritance-and-Composition/
https://dev-cool.tistory.com/22
https://zangzangs.tistory.com/44