# ITEM 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 사용하려고 설계한 것이 아니기 때문에 인스턴스화를 막아야 한다.

## 생성자를 명시하지 않기?

생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다.

## 추상 클래스로 만들기?

abstract 클래스는 인스턴스로 생성하는 것이 불가능하지만 하위 클래스를 만들어 인스턴스화하면 그만이다.

## 인스턴스화 막기

private 생성자 추가한다.

```java
class DateUtility {
    private DateUtility() {
        /**
         * 클래스 내부에서도 호출이 안되도록 막는다.
         */
        throw new AssertionError();
    }

    // 생략
}

```