# ITEM 28. 배열보다는 리스트를 사용하라

## 공변과 불공변 

> 공변과 불공변?
> 
> 공변: 배열, 공변성이란 자신이 상속받은 부모 객체로 타입을 변환시킬 수 있다는 것을 뜻한다. Sub라는 클래스가 Super라는 클래스의 하위클래스라고 할때, 배열 Sub[]는 Super[]의 하위타입이 된다.
> 
> 불공변: 제네릭, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.

```java
Object[] objectArray = new Long[1];
objectArray[0] = "저장 안되는 문자열"; // ArrayStoreException을 던진다.
```

위의 코드는 런타임에 오류가 난다.

```java
List<Object> o1 = new ArrayList<Long>(); // 호환되지 않는 타입
o1.add("저장 안되는 문자열");
```

위의 코드는 컴파일 시에 오류가 난다. 

## 실체화와 소거

배열 

```java
// .java
public static void main(String[] args) {
          String[] strings = new String[3];
        strings[0] = "1";
        strings[1] = "2";
        strings[2] = "3";
}

// .class
public static void main(String[] var0) {
    String[] var1 = new String[]{"1", "2", "3"};
}
```

제네릭을 사용한 리스트

```java
// .java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    strings.add("1");
    strings.add("2");
    strings.add("3");

    System.out.println("hello = " + strings);
}

// .class
public static void main(String[] var0) {
    ArrayList var1 = new ArrayList(); // 타입정보가 사라졌다?
    var1.add("1");
    var1.add("2");
    var1.add("3");
    System.out.println("hello = " + var1);
}
```

배열은 런타임 시점에 타입을 인지하고 실행하는데(실체화) 반해 제네릭은 런타임 시점에 타입 정보를 소거한다.

이처럼 배열과 제네릭은 잘 어우러지지 못한다. 예를 들어 new List<E>[], new List<String>[], new E[] 이런 식으로 작성할 수 없다.

## 왜 배열보다 리스트를 사용해야할까?

위와 같은 이유로 배열과 제네릭은 함께 쓰이기 어렵다.

그러면 둘 중 하나를 써야하는데 리스트를 써야 하는 이유는 타입에 더 안전하며 런타임이 아닌 컴파일 시점에 에러를 잡아주기 때문이다.
