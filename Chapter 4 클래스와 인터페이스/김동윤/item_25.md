# 톱레벨 클래스는 한 파일에 하나만 담긔
## 톱레벨 클래스?
- 톱 레벨 클래스는 중첩 클래스가 아닌 클래스이다.
- 우리가 일반적으로 사용하는 단일 형태의 클래스

## 한 파일에 여러개의 톱레벨 클래스가 담긴 것?

- 바로 중첩되지 않은 클래스가 하나의 java 파일에 여러개 담긴 것

```java
class A {
    String NAME = "a";
}

class B{
    String NAME = "b";
}
```

## 위처럼 하면 일어날 수 있는 컴파일 에러 상황!
톱레벨 클래스 중복정의
#### Utensil과 Dessert를 참조하는 Main 클래스

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```


#### Utensil 파일에 함께 선언된 Utensil클래스와 Dessert 클래스

```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```
- Main을 실행하면 pancake를 출력한다.


#### Dessert 파일에 함께 선언된 Utensil클래스와 Dessert 클래스

- 똑같은 두 클래스를 담은 Dessert.java파일
```java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```
- Utensil파일에 있는 똑같은 클래스를 중복 정의했을때 컴파일러의 동작 결과가 달라지므로 문제가 발생한다.

#### 컴파일 오류 발생 명령어

- javac Main.java Dessert.java // 컴파일 오류 발생
- 컴파일러는 가장 먼저 Main.java를 컴파일하고, 그 안에서 Utensil.java파일을 살펴 Utensil 클래스와 Dessert 클래스를 찾아낼 것이다.

- 그리고 Dessert.java를 처리할 때 같은 클래스들의 정의가 이미 있음을 알게 됨

#### 정상 동작 명령어
```

javac Main.java // pancake 출력
javac Main.java Utensil.java // pancake 출력
javac Dessert.java Main.java // potpie 출력
```
- 이렇게 컴파일러에 어느 소스파일을 먼저 건네느냐에 따라서 정상 동작할수도 또는 컴파일 오류가 발생 가능
- 따라서 이런 문제가 발생하지 않게 바로잡기 필요

## 결론
- 한 개의 파일에 한 개의 톱레벨 클래스만!
- 정적 멤버 클래스 방식 사용
- =>  컴파일러가 한 클래스에 대한 정의를 여러 개 만들어내는 일이 사라진다.
- => 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.
```java
public class AB {
    private static class A {

    }

    private static class B {

    }
}
```

## 출처 
https://github.com/Meet-Coder-Study/book-effective-java
https://velog.io/@alkwen0996/