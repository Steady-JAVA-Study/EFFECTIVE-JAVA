# ITEM 65. 리플렉션보다는 인터페이스를 사용하라

> Reflection ?
> 
> 실행 중인 Java 프로그램이 자체적으로 검사하거나, 수정할 수 있는 기능.
>
> 리플렉션(java.lang.reflect)을 통해 프로그램에서 임의의 클래스에 접근할 수 있다. Class 객체가 주어지면 그 클래스의 생성자, 그 주어진 그 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Filed 인스턴스를 가져올 수 있고, 이 인스턴스들로 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다.
>
> Constructor, Method, Field 인스턴스를 이용해 실제 생성자, 메서드 필드를 조작할 수도 있다. 인스턴스를 생성하거나, 메서드를 호출하거나, 필드에 접근할 수 있다는 뜻이다.

## Reflection의 단점

1. 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다. 예외 검사도 마찬가지고, 런타임 오류가 생길 수도 있다.
2. 코드가 지저분해지고, 장황해진다.
3. 성능이 떨어진다.

## Reflection을 사용할 때

컴파일타임에 이용할 수 없는 클래스를 사용해야 하는 경우 인스턴스 생성에만 쓰고, 인스턴스는 인터페이스나, 상위 클래스로 참조해 사용하자

```java
public class Main {
    // 코드 65-1 리플렉션으로 생성하고 인터페이스로 참조해 활용한다. (372-373쪽)
    public static void main(String[] args) {
 
 
        // 클래스 이름을 Class 객체로 변환
        Class<? extends Set<String>> cl = null;
        try {
            cl = (Class<? extends Set<String>>)  // 비검사 형변환!
                    Class.forName("java.util.HashSet");
        } catch (ClassNotFoundException e) {
            fatalError("클래스를 찾을 수 없습니다.");
        }
 
        // 생성자를 얻는다.
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
        }
 
        // 집합의 인스턴스를 만든다.
        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("생성자에 접근할 수 없습니다.");
        } catch (InstantiationException e) {
            fatalError("클래스를 인스턴스화할 수 없습니다.");
        } catch (InvocationTargetException e) {
            fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
        } catch (ClassCastException e) {
            fatalError("Set을 구현하지 않은 클래스입니다.");
        }
 
        // 생성한 집합을 사용한다.
        s.addAll(Arrays.asList("a","b","c").subList(1, 3));
        System.out.println(s);
    }
 
    private static void fatalError(String msg) {
        System.err.println(msg);
        System.exit(1);
    }
}
```

위 코드의 단점 2가지

1. 런타임에 총 6가지나 되는 예외를 던질 수 있다.
2. 클래스 이름만으로 인스턴스를 생성해내기 위해 무려 25줄이나 되는 코드를 작성했다.

---

## private 생성자 접근

```java
public class ReflectionTest {
    static class Person {

        private String name;
        private Integer age;

        private Person(String name, Integer age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public Integer getAge() {
            return age;
        }
    }
    public static void main(String[] args) throws Exception{

        Constructor<Person> constructor = Person.class.getDeclaredConstructor(String.class, Integer.class);

        constructor.setAccessible(true);

        Person person = constructor.newInstance("yoon", 26);

        System.out.println(person.getName());
        System.out.println(person.getAge());

    }
}
```
