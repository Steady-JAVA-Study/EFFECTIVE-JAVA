# ITEM 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단인 public 생성자 이외에 정적 팩터리 메서드(static factory method, `public static`)를 통해 인스턴스를 획득할 수 있다.

```java

public class Book {
    private String name;
    private String author;
    private String publisher;

    public static createBook() {
        return (Book 인스턴스);
    }
}

```

## 정적 패터리 메서드

### 장점

#### 1. 이름을 가질 수 있다.

정적 팩터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다. 

```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    // 생성자를 이용한 객체 초기화
    public Book(String name) {
        this.name = name;
    }

    // 펙토리 메서드를 이용한 객체 초기화
    public static createBookWithName(name) {
        return new Book(name);
    }
}
```

```
Book book1 = new Book("effeciveJava"); 
// "effectiveJava" 가 어떤 인스턴스 변수인지 알기 어렵다.
        
Book book2 = Book.createBookWithName("effectiveJava"); 
// 이름이 "effectiveJava" 임을 한눈에 알 수 있다.
```

#### 2. 하나의 시그니처로 여러가지 객체를 생성할 수 있다.

매개변수의 타입과 갯수가 같은 생성자를 여러개 만들 수 없다.

```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    public Book(String name) {
        this.name = name;
    }
    
    // 불가능
    public Book(String author) {
        this.author = author;
    }
}
```

하지만 아래와 같이 정적 팩토리 매서드에서는 가능하다.

```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    public static Book createBookWithName(String name) {
        Book book = new Book();
        book.name = name;
        return book;
    }

    public static Book createBookWithAuthor(String author) {
        Book book = new Book();
        book.author = author;
        return book;
    }
}
```

#### 3. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
    ...
```

Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않는다. 따라서 (특히 생성 비용이 큰) 같은 객체가 자주 요청되는 상황이라면 성능이 올라간다.

정적 팩토리 메서드로 객체 안에 미리 정의된 static final 상수 객체를 반환

반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다. -> `인스턴스 통제(instance-controlled) 클래스`

> 인스턴스를 통제하는 이유는?
> 
> 인스턴스를 통제하면 클래스를 싱글턴(singleton; 아이템3)으로 만들 수도, 인스턴스화 불가(noninstantiable; 아이템 4)로 만들수도 있다.
> 또한, 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.(a == b일 때만 a.equals(b)가 성립)

#### 4. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

리턴 타입은 인터페이스로 지정하고, 실제로는 인터페이스의 구현체를 리턴함으로서 구현체의 API 는 노출시키지 않고 객체를 생성할 수 있다.

```java
public interface Car {

  ...
}

```

```java

public class ElectricCar implements Car {
    int position = 0;
    
    public static Car create(int position) {
        return new ElectricCar(position);
    }
    ....
}
```

API를 만들 떄 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.

예를 들어 java.util.Collection에서 정적 팩터리 메서드를 통해 45개의 유틸리티 구현체를 제공한다.

인터페이스만 노출되므로 '개념적인 무게 줄이기' 가능. 프로그래머가 알아야하는 개수와 난이도가 줄어든다.

자바 8부터는 public static 메서드를 인터페이스 추가 가능하다. 그래서 굳이 Collecions라는 클래스를 만들지 않고도 인터페이스에 구현 가능하다. 하지만 private static은 자바 9부터 가능하다. 자바9를 쓰면 private static까지 인터페이스에 추가할 수 있으니 java.util.Collection 같은 유틸성 클래스들은 필요 없어진다.

#### 5. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

장점 4의 연장선으로 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

```java
public class Level {
  ... 
  public static Level of(int score) {
    if (score < 50) {
      return new Basic();
    } else if (score < 80) {
      return new Intermediate();
    } else {
      return new Advanced();
    }
  }
  ...
```

#### 6. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

인터페이스나 클래스가 만들어지는 시점에서 하위 타입의 클래스가 존재하지 않아도 나중에 만들 클래스가 기존의 인터페이스나 클래스를 상속 받으면 언제든지 의존성을 주입 받아서 사용가능하다. 반환값이 인터페이스가 되며 정적 팩터리 메서드이 변경없이 구현체를 바꿔 끼울 수 있다.

**다른 누군가가 구현하고 있는 코드**

```java
public class ElectricCar extends Car {
    ...
}
```

```java
ublic class Car {

  	public static Car getCar(int position) {
        Car car = new Car();
        
        // Car 구현체의 FQCN(Full Qualified Class Name)을 읽어온다
        // FQCN에 해당하는 인스턴스를 생성한다.
        // car 변수를 해당 인스턴스를 가리키도록 수정
        
        return car;
    }
}
```

`getCar`는 실행시점에 위 코드에 뭐가 적혀있냐에 따라 다르게 작동한다.

### 단점

#### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

`java.util.Collections`는 상속할 수 없다.

상속보다 컴포지션을 사용(아이템 18)하도록 유도하고 불변 타입(아이템 17)으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있다.

#### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자와는 달리 Javadoc 문서에서 따로 모아주는 기능이 없다.

> JavaDoc? 
> 
> docs.oracle.com/javase/8/docs/api/index.html
> 
> 위와 같은 형태의 html 문서를 JavaDoc이라고 한다.
> 
> https://creampuffy.tistory.com/81 -> 인텔리제이로 JavaDoc 생성

생성자처럼 API 설명에 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.


