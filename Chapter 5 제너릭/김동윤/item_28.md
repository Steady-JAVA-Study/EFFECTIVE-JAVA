.# 배열보다는 리스트를 사용하라
## 1. 배열은 공변 (함께 변함 )
상위 클래스(Super)가 있고 이를 상속하는 하위 클래스(Sub)가 있습니다.

```java
Super sub = new Sub();
```
배열에서의 공변은 아래처럼 배열에서도 함께 변해서 적용된다는 뜻입니다.

```java
Super[] sub = new Sub[]{};
```

Ex)

```java
Object[] objects = new Long[1];
objects[0] = "나는 String, 원랜 들어가면 안되지 ";
```

- Long은 Object를 상속
- 배열로 사용되고 있어서 런타임에는 실패하
- 컴파일 에러는 뜨지 않음
- 실제로 실행 => ArrayStoreException


불공변은 반대로 같이 변하지 않는 것입니다. 즉, 관계가 유지되지 않습니다. 바로 제네릭이 불공변의 특성을 가지고 있습니다.

List<Super> subs = new ArrayList<Sub>(); // 같은 타입이 아니므로 컴파일 에러!
위 코드처럼 Sub가 Super를 상속하더라도 같은 타입이 아니면 컴파일 에러가 뜹니다.

Ex)
```java
List<Object> objects = new ArrayList<Long>(); // 컴파일 에러!
```
  
## 2. 배열은 실체화된다.

  
-  배열은 런타임 시점에 타입을 인지하고 실행 (실체화) 
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
    String[] var1 = new String[]{"1", "2", "3"}; //TYPE HERE
}
```
  
-  제네릭은 런타임 시점에 타입 정보를 소거(erasure)
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
    ArrayList var1 = new ArrayList(); // 타입정보가 사라짐(런타임 시점에 없어짐)
    var1.add("1");
    var1.add("2");
    var1.add("3");
    System.out.println("hello = " + var1);
}
```
  
-  제네릭의 타입 시스템 자체가 컴파일 시점에 에러를 잡고 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 것이므로! 
  
## 결론
-  배열과 제네릭은 함께 쓰이기 NOT GOOD '
- THEN 둘 중 하나를 써야하는데 리스트를 써야 하는 이유? => 타입에 더 안전하며 런타임이 아닌 컴파일 시점에 에러를 잡아줘서
  
## 출처 
https://github.com/Meet-Coder-Study/book-effective-java/blob/main/5%EC%9E%A5/28_%EB%B0%B0%EC%97%B4%EB%B3%B4%EB%8B%A4%EB%8A%94_%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC_%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%EC%9D%B4%ED%98%B8%EB%B9%88.md