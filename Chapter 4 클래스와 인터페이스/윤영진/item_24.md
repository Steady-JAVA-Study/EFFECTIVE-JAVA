# ITEM 24. 멤버 클래스는 되도록 static으로 만들라

> 중첩 클래스? - Member Class
> 
> 다른 클래스 안에 정의된 클래스, 자신을 감싼 바깥 클래스에서만 사용되어야 한다.
> 
> 종류: 정적 멤버 클래스, 비정적 멤버 클래스, 익명 클래스, 지역 클래스

## 중첩 클래스를 사용하는 이유 

### 정적 멤버 클래스

class 내부에서 static으로 선언된 클래스

바깥 클래스의 private 멤버에도 접근 가능

private로 선언 시 바깥 클래스에서만 접근 가능


```java
public class Animal {
    private String name = "cat";

    // 열거 타입도 암시적 static
    public enum Kinds {
        MAMMALS, BIRDS, FISH, REPTILES, INSECT
    }

    private static class PrivateSample {
        private int temp;

        public void method() {
            Animal outerClass = new Animal();
            System.out.println("private" + outerClass.name); // 바깥 클래스인 Animal의 private 멤버 접근
        }
    }

    public static class PublicSample {
        private int temp;

        public void method() {
            Animal outerClass = new Animal();
            System.out.println("public" + outerClass.name); // 바깥 클래스인 Animal의 private 멤버 접근
        }
    }
}

public class Calculator {
    public enum Operation { // 열거 타입도 암시적 static 이다.
        PLUS, MINUS, MULTIPLE, SUBTRACT
    }
}
```

```java
void test() {
    Animal animal = new Animal();
    new Animal.PublicSample(); // O
    new Animal.PrivateSample(); // X
}
```

바깥 클래스가 표현하는 객체의 한 부분(구성요소)일 때 사용한다. 
- ex) Map의 Entry

### 비정적 멤버 클래스


static이 붙지 않은 멤버 클래스

비정적 멤버 클래스의 인스턴스 메서드에서 클래스명.this를 사용해 바깥 인스턴스의 메서드나 참조 가져올 수 있다.

바깥 인스턴스 없이는 생성 불가

```java
public class TestClass {
    private String name = "yeonlog";

    public class PublicSample {
        public void printName() {
            // 바깥 클래스의 private 멤버 가져오기
            System.out.println(name);
        }

        public void callTestClassMethod() {
            // 바깥 클래스의 메소드 호출하기
            TestClass.this.testMethod();
        }
    }

    public PublicSample createPublicSample() {
        return new PublicSample();
    }

    public void testMethod() {
        System.out.println("hello world");
    }
}
```

```java
TestClass test = new TestClass();
test.new PublicSample(); // 바깥 instance 없이는 생성할 수 없다 -> 메모리 낭비
PublicSample publicSample1 = test.createPublicSample(); // 바깥 클래스의 인스턴스 메서드에서 생성자 호출
PublicSample publicSample2 = test.new PublicSample(); // 바깥 인스턴스 클래스.new 멤버클래스()
```

1. 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자 호출
2. 바깥 클래스 인스턴스.new 멤버클래스() 호출
   - 위 두 방법으로 바깥 클래스 - 비정적 멤버 클래스의 관계 생성됨
   - 관계 정보는 비정적 멤버 클래스의 인스턴스 내에서 메모리 공간을 차지

### 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 static을 붙이자

바깥 인스턴스 - 멤버 클래스 관계를 위한 시간과 공간 소모

Garbage Collection이 바깥 클래스의 인스턴스 수거 불가 -> 메모리 누수 발생

참조가 눈에 보이지 않아 개발이 어려움

## 익명 클래스 

이름이 없는 클래스

바깥 클래스의 멤버가 아니다

쓰이는 시점에 선언 + 인스턴스 생성

```java
public class Calculator {
    private int x;
    private int y;

    public Calculator(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int plus() {
        Operator operator = new Operator() {
            private static final String COMMENT = "더하기"; // 상수
            // private static int num = 10; // 상수 외의 정적 멤버는 불가능
          
            @Override
            public int plus() {
                // Calculator.plus()가 static이면 x, y 참조 불가
                return x + y;
            }
        };
        return operator.plus();
    }
}

interface Operator {
    int plus();
}
```

즉석에서 작은 함수 객체나 처리 객체를 만드는데 주로 사용했으나 람다 등장 이후로 람다가 역할을 대체

```java
List<Integer> list = Arrays.asList(10, 5, 6, 7, 1, 3, 4);

// 익명 클래스 사용
Collections.sort(list, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
});

// 람다 도입 후
Collections.sort(list, Comparator.comparingInt(o -> o));
```

정적 팩터리 메서드 구현 시 익명 클래스 사용

```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requiredNonNull(a);
    
    return new AbstracktList<>() {
        @Override public Integer get(int i) {
            return a[i];
        }
    }
}
```

### 지역 클래스 

지역 변수를 선언할 수 있는 곳이면 어디서든 선언 가능

유효 범위는 지역 변수와 동일

주로 메서드 바디에 선언하여 사용   

```java
public class TestClass {
    private int number = 10;

    public TestClass() {
    }

    public void foo() {
        // 지역변수처럼 선언
        class LocalClass {
            private String name;
            // private static int staticNumber; // 정적 멤버 가질 수 없음

            public LocalClass(String name) {
                this.name = name;
            }

            public void print() {
                // 비정적 문맥에선 바깥 인스턴스를 참조 가능
                // foo()가 static이면 number에서 컴파일 에러
                System.out.println(number + name);
            }
        }
        LocalClass localClass1 = new LocalClass("local1"); // 이름이 있고
        LocalClass localClass2 = new LocalClass("local2"); // 반복해서 사용 가능
    }
}
```

---

메서드 밖에서 사용할 것이다 -> member class
- 그 중 member class가 바깥 instance를 참조한다. -> non-static
- 그 외 -> static

딱 한 곳에서 사용 && 사전 struct가 있다. -> 익명 클래스
그 외 -> 지역 클래스
