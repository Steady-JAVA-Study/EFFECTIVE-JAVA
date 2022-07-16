# ITEM 10. equals는 일반 규약을 지켜 재정의하라

equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어서 자칫하면 끔찍한 결과를 초래한다.

아래 열거한 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선이다.

## 1. 각 인스턴스가 본질적으로 고유하다.

값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스 -> `ex. Thread`

## 2. 인스턴스의 '논리적 동치성'을 검사할 일이 없다.

> 물리적 동치?
> 
> 두 객체가 메모리의 같은 주소를 가리키고 있는가를 뜻한다.

> 논리적 동치성 ?
> 
> p와 q의 진릿값이 서로 같은 경우, p = q 

```java
public class Student {

    // 학생 번호
    private int studentId;

    public Student(int studentId) {
        this.studentId = studentId;
    }

    // equals 재정의 : 학생 번호가 같으면 true
    @Override
    public boolean equals(Object o) {
        if(o instanceof Student){
            Student student = (Student) o;
            return studentId == student.studentId;
        }
        return false;
    }
}
```

위의 Student 객체는 생성될 때마다 물리적으로는 다르겠지만, 학생 번호만 같다면 논리적으로 같다.


## 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우

Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로부터, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다.

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
	public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof List))
            return false;

        ListIterator<E> e1 = listIterator();
        ListIterator<?> e2 = ((List<?>) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        return !(e1.hasNext() || e2.hasNext());
    }
}
```

## 4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없을 경우

equals가 실수로라도 호출되는 걸 막고 싶다면 다음처러 구현하자.

```java
@Override
public boolean equals (Object object) {
	throw new AssertionError();
}
```

## equals를 재정의해야 하는 경우 

객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.

주로 값 클래스(String, Integer, ...)들의 여기 해당한다. 두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것이다.


## equals 재정의의 5가지 일반 규약

### 1. 반사성 (reflexivity)

객체는 자기 자신과 같아야 한다. -> `x.equals(x)는 true`

### 2. 대칭성 (symmetry)

`x.equals(y)`가 참이면 `y.equals(x)`도 참이어야 한다.

```java
public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = s;
    }

    @Override
    public boolean equals(Object o) {
        // ...생략
        if (o instanceof String) {  //String과의 비교 연산 시도
            return s.equalsIgnoreCase((String) o);
        }
        // ...생략
        return false;
    }
}
```

```java
CaseInsensitiveString cis = new CaseInsensitiveString("AAA");
String s = "aaa";

cis.equals(s); //true
s.euqals(cis); //false
```

CaseInsensitiveString의 equals 자체는 잘 작동하지만 그 반대는 작동하지 않는다.
String의 equals는 CaseInsensitiveString의 존재를 알지 못하기 때문이다.

이를 해결하기 위해서는 equals에서 일반 String과 비교하는 부분을 없애야 한다.

```java
@Override
public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
        }
```

### 3. 추이성 (transitivity)

`x.equals(y)`가 true이고, `y.equals(z)`가 true이면 `x.equals(z)`도 true이다.

```java
public class Point {

    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}
```

```java
public class ColorPoint extends Point {

    private final Color color;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.color == ((ColorPoint) o).color;
    }
}
```

```java
ColorPoint a = new ColorPoint(2, 3, Color.RED);
Point b = new Point(2, 3);
ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

System.out.println(a.equals(b)); // true
System.out.println(b.equals(c)); // true
System.out.println(a.equals(c)); // false
```

위의 방식은 대청싱은 지켜주지만, 추이성을 깨버린다.

#### 상속 대신 컴포지션(아이템 18)을 사용해라. 

아쉽게도, 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

다만 컴포지션 패턴을 통해 이를 우회할 수 있다.

```java
public class ColorPoint {

    private Point point;
    private Color color;

    public ColorPoint2(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
```

> 컴포지션 ?
> 
> 다른객체의 인스턴스를 자신의 인스턴스 변수로 포함해서 메서드를 호출하는 기법


## 4. 일관성(consistency)

두 객체가 같다면(어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻이다.

**equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.**

```java
public final class URL implements java.io.Serializable {

	public boolean equals(Object obj) {
        if (!(obj instanceof URL))
            return false;
        URL u2 = (URL)obj;

        return handler.equals(this, u2);
    }
}
```

```java
public abstract class URLStreamHandler {
	protected boolean equals(URL u1, URL u2) {
        String ref1 = u1.getRef();
        String ref2 = u2.getRef();
        return (ref1 == ref2 || (ref1 != null && ref1.equals(ref2))) &&
               sameFile(u1, u2);
    }
	protected boolean sameFile(URL u1, URL u2) {
        // Compare the protocols.
        if (!((u1.getProtocol() == u2.getProtocol()) ||
              (u1.getProtocol() != null &&
               u1.getProtocol().equalsIgnoreCase(u2.getProtocol()))))
            return false;

        // Compare the files.
        if (!(u1.getFile() == u2.getFile() ||
              (u1.getFile() != null && u1.getFile().equals(u2.getFile()))))
            return false;

        // Compare the ports.
        int port1, port2;
        port1 = (u1.getPort() != -1) ? u1.getPort() : u1.handler.getDefaultPort();
        port2 = (u2.getPort() != -1) ? u2.getPort() : u2.handler.getDefaultPort();
        if (port1 != port2)
            return false;

        // Compare the hosts.
        if (!hostsEqual(u1, u2))
            return false;

        return true;
    }

	protected boolean hostsEqual(URL u1, URL u2) {
        InetAddress a1 = getHostAddress(u1);
        InetAddress a2 = getHostAddress(u2);
        // if we have internet address for both, compare them
        if (a1 != null && a2 != null) {
            return a1.equals(a2);
        // else, if both have host names, compare them
        } else if (u1.getHost() != null && u2.getHost() != null)
            return u1.getHost().equalsIgnoreCase(u2.getHost());
         else
            return u1.getHost() == null && u2.getHost() == null;
    }
}
```

위 코드와 같이 같은 URL인지를 검사할 때 IP 주소로 변경해 비교하게 된다. 이때 호스트 이름을 IP주수로 변경하는 것이 네트워크를 통해야 하는데 그 결과가 항상 같다고 보장할 수 없다.

이러한 문제를 피하기 위해 항상 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만을 수행해야 한다는 것을 알 수 있다.

## 5. non-null

`x.equals(null) == false;`, 모든 객체가 null과 같지 않아야 한다는 뜻이다.

형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사한다면 묵시적 null 검사를 할 수 있다.

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Member)) {
        return false;
    }
    Member member = (Member)o;

    return email.equals(member.email) && name.equals(member.name) && age.equals(member.age);
}
```

instanceof는 첫 번째 피연산자가 null이면 false를 반환한다. 따라서 입력이 null이면 타입 확인 단계에서 false를 반환하기 때문에 null 검사를 명시적으로 하지 않아도 된다.

## equals 메서드 구현 방법 단계별 정리

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

## 전형적인 equals 메서드의 예

````java
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;
    if (!(o instanceof Member))
        return false;
    Member member = (Member)o;
    return email.equals(member.email) && name.equals(member.name) && age.equals(member.age);
}
````