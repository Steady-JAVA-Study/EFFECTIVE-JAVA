# ITEM 25. 톱레벨 클래스는 한 파일에 하나만 담으라

> 톱레벨 클래스?
> 
> 함수, 클래스 또는 다른 무언가로 감싸지지 않은 모든 구문은 톱레벨로 간주된다.
> 
> 즉, 톱레벨 클래스란 중첩 클래스가 아닌 것을 말한다.

```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

실행해도 class가 중복 된다는 에러와 함께 실행되지 않는다.

여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스르 사용하는 방법을 고민해보자.

