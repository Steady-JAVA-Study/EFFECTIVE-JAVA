# ITEM 20. 추상 클래스 보다는 인터페이스를 우선하라 

## 추상 클래스와 인터페이스

### 공통점

인스턴스로 생성이 불가능하다. 

선언부만 있는 추상 메서드를 갖는다.
- 인스턴스 메서드를 구현 형태로 제공할 수 있다.


### 차이점

#### 1. 목적 

추상 클래스
- 추상 클래스를 상속받아 기능을 이용하고 확장시켜야한다.

인터페이스
- 구현을 강제해서 구현 객체가 같은 동작을 하도록 보장한다.

#### 2. 상속

추상 클래스
- 단일 상속만 지원하기 때문에, **다중 클래스 상속이 불가능**하다.

```java
// Amy에 Student을 추가로 상속 받고 싶으면? -> 불가능!
public class Amy extends Person {
  // ...
}

abstract class Student {
  // ...
}
```

인터페이스
- 구현해야하는 메서드만 구현하면 어떤 클래스를 상속하던 간에 같은 타입으로 취급한다.
- **여러 인터페이스를 상속이 가능**하다.

```java
// Amy에 Student을 추가로 상속 받고 싶으면? -> implements 구문만 추가하면 됨!
public class Amy extends Person implements Student {
  // ...
    
}

interface Student {
  // ...
}
```

#### 3. 기존 클래스의 확장

추상 클래스 
- 기존 클래스에 끼워넣기 어렵다.
```java
// 만약 여기서 추상 클래스 Person을 확장해야한다면?
public class Amy extends Student {
  // ...
}

public class Jason extends Student {
  // ...
}

abstract class Student {
	// ...
}

// 이렇게??
public abstract class Person {
  public abstract void walk();
}

abstract class Student extends Person {
	// ...
}

// Student를 상속 받았던 객체들은 모두 walk()이라는 메소드의 구현을 해야한다.
public class Amy extends Student {
  // ...
}
```

인터페이스
- 손쉽게 인터페이스를 구현해 넣을 수 있다.

```java
// 만약 여기서 추상 클래스 Person을 확장해야한다면?
public class Amy extends Student {
  // ...
}

public class Jason extends Student {
  // ...
}

abstract class Student {
	// ...
}

// 원하는 객체만 구현을 할 수 있다.
public class Amy extends Student implements Person {
  // ...
}
```

#### 4. 믹스인 정의

> 믹스인?
> 
> 특정 클래스의 주 기능에 추가적인 기능을 혼합하는 것이다.

추상클래스
- 단일 상속만 지원하기 때문에 믹스인이 들어갈만한 자리가 없다.

```java
// 만약 Comparable이 추상 클래스라면?
public abstract class Comparable<T> {
  public abstract int compareTo(T o);
}

// 들어갈 자리가 없다! 
public class Amy extends Student {
	// ...
}
```
인터페이스
- 손쉽게 구현이 가능하다.


```java
public class Amy extends Student implements Comparable {

  @Override
  public int compareTo(Object o) {
    return 0;
  }
}
```


#### 5. 계층구조

Amy라는 객체가 Student인 동시에 Intern인 경우

추상클래스
- 복잡한 계층구조일 경우에 많은 조합이 필요하고, 고도 비만 계층으로 이어질 수 있다.

```java
public abstract class Student {
    // ...
}

public abstract class Intern {
    // ...
}

// 단일 상속만 가능하기 때문에 인턴인 동시에 학생인 추상 클래스가 필요하다.
public abstract class StudentIntern {
    // ...
}

public class Amy extends SutdentIntern {
	// ...
}
```

인터페이스 
- 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```java
public interface Student {
    // ...
}

public interface Intern {
    // ...
}

// 문제 없다.
public class Amy implements Student, Intern {
	// ...
}
```

## 인터페이스와 추상골격 구현 클래스 - 템플릿 메서드 패턴

인터페이스 + 골격 구현 클래스

추상 클래스처럼 구현을 도와주는 동시에, 추상클래스로 타입을 정의할 때 따라오는 심각한 제약에서 자유롭다.

```java
// 인터페이스
public interface Phone {
  void booting();
  void greeting();
  void shutdown();
  void process();
}
```
```java
public class IPhone implements Phone {

  @Override
  public void booting() {
    System.out.println("booting ...");
  }

  @Override
  public void greeting() {
    System.out.println("I am iPhone");
  }

  @Override
  public void shutdown() {
    System.out.println("shut down ...");
  }

  @Override
  public void process() {
      booting();
      greeting();
      shutdown();
  }
}
```

```java
public class GalaxyPhone implements Phone {
  @Override
  public void booting() {
    System.out.println("booting ...");
  }

  @Override
  public void greeting() {
    System.out.println("I am galaxy phone");
  }

  @Override
  public void shutdown() {
    System.out.println("shut down ...");
  }

  @Override
  public void process() {
    booting();
    greeting();
    shutdown();
  }
}
```

iPhone과 GalaxyPhone 모두 같은 인터페이스(Phone)를 구현하고 있고, booting()과 shutdown()은 같은 동작을 하고 있다.

여기서 추상골격 구현 클래스을 이용하면 중복 코드를 제거할 수 있다.

```java
// 추상골격 구현 클래스 (보통 Abstract~의 네이밍을 사용한다)
public abstract class AbstractPhone implements Phone {

	// 같은 동작을 하는 메소드를 여기에 정의한다.
  @Override
  public void booting() {
    System.out.println("booting ...");
  }

  @Override
  public void shutdown() {
    System.out.println("shut down ...");
  }

  @Override
  public void process() {
    booting();
    greeting();
    shutdown();
  }
}
```

```java
public class IPhone extends AbstractPhone implements Phone {

  @Override
  public void greeting() {
    System.out.println("I am iPhone");
  }
}
```


```java
public class GalaxyPhone extends AbstractPhone implements Phone {

  @Override
  public void greeting() {
      System.out.println("I am galaxy phone");
  }
}
```
