# private 생성자나 열거 타입으로 싱글턴임을 보증하라

## SINGLETON : 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 한번의 객체 생성으로 재사용이 가능해져 메모리 낭비를 방지
- 전역성을 갖기 때문에 다른 객체와 공유가 용이
- 클라이언트를 테스트하기 어려움

## 싱글톤 만들기
### 1) public static final 필드 방식
```java
public class Printer {
    public static final Printer INSTANCE = new Printer();
    private Printer() { ... }

    public void print() { ... }
}
```

- public, protected 생성자가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나 뿐임이 보장

### 2) 정적 팩터리 방식
```java
public class Printer {
    private static final Printer INSTANCE = new Printer();
    private Printer() { ... }
    public static Printer getInstance() { return INSTANCE; }

    public void print() { ... }
}
```
- Printer.getInstance는 항상 같은 객체의 참조를 반환


### public static final / 정적팩토리 방식으로 만든 싱글톤의 직렬화!?
- 이때의 싱글톤 클래스 직렬화(자바 객체 -> 바이트 코드) 하려면 단순 Serializable 선언하는 것만으로는 X
- 모든 인스턴스 필드에 transient를 선언하고, readResolve 메서드를 제공해야만 역직렬화시(바이트 코드 -> 자바 객체) 에 새로운 인스턴스가 만들어지는 것을 방지 (즉 역직렬화 돼서 새 객체가 생기는 것이 아니라 기존 객체를 사용하도록 readResolve 메소드 선언해야 함)
```
private Object readResolve() {
    return INSTANCE;
}
```
## ?? transient 인스턴스에 선언한다는 것이 무엇이니?  & 직렬화에서 싱글턴 보장

### transient : 직렬화 과정에서 필드의 상태값을 전송하지 않는다는 키워드
> - Serialize하는 과정에 제외하고 싶은 경우 선언
- 패스워드와 같은 보안정보가 직렬화(Serialize) 과정에서 제외하고 싶은 경우에 적용
- 다양한 이유로 데이터를 전송을 하고 싶지 않을 때 선언
![](https://velog.velcdn.com/images/myway00/post/d99595ab-9151-4006-a43d-3fa177ff1b29/image.png)
[사진 및 설명 출처](https://nesoy.github.io/articles/2018-06/Java-transient)

- 역직렬화 시 반환되는 인스턴스를 관장하는애는 readResolve 메서드


이 클래스는 바깥에서 생성자를 호출하지 못하게 막는 방식으로 인스턴스가 오직 하나만 만들어짐을 보장

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
}
```
=> BUT implements Serializable 을 추가하는 순간 더 이상 싱글턴이 아님
- 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환

> **readResolve** 기능을 이용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체 가능

```java
//인스턴스 통제를 위한 readResolve - 개선의 여지가 있다
private Object readResolve() {
  //진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
  return INSTANCE;
}
```

- 이 메서드는 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 Elvis 인스턴스를 반환한다.

- 따라서 Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니 모든 인스턴스 필드를 transient로 선언

### 그래서 transient로 왜 선언 ? 
- readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다고 한다

- 아니면 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다고 한다. 

- readResolve를 읽기 전에 (기존 인스턴스 반환 전에) 엘비스 공격과 같이 readResolve 수행 전 실행해서, 가비지 콜렉터가 방금 역직렬화되면서 만들어진 객체를 버리기 전에 이를 채가서 엘비스에 대입할 수 있다, 

- The elvis stealing attack steals the freshly created Elvis before it's readResolve method (= it's unresolve) had a chance to replace (resolve) it.

### 3) 열거 타입 방식의 싱글턴
```java
public enum Printer {
    INSTANCE;

    public void print() { ... }
}
```
- 간결하고, 추가 노력 없이 직렬화 가능해서 이것을 권장 O
- 전통적인 싱글턴보다 better

## 출처
이펙티브 자바 독서
https://github.com/Java-Bom/ReadingRecord/issues/4

https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/11/12/%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-%EC%88%98%EB%A5%BC-%ED%86%B5%EC%A0%9C%ED%95%B4%EC%95%BC-%ED%95%9C%EB%8B%A4%EB%A9%B4-readResolve-%EB%B3%B4%EB%8B%A4%EB%8A%94-%EC%97%B4%EA%B1%B0-%ED%83%80%EC%9E%85%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90/

https://stackoverflow.com/questions/37660696/elvisstealer-from-effective-java