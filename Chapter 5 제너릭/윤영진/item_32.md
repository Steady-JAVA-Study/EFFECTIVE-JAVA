# ITEM 32. 제네릭과 가변인수를 함께 쓸 때는 신중해라

### 가변인수 ?

```java
public static void display(String ...infos) {
    for (String info: infos) {
        System.out.println(s);
    }
}
```

가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해준다.

## 가변인수와 제너릭 

가변인수(varargs) 메서드와 제네릭은 자바 5 때 함께 추가되었지만 잘 어울리지 않는다. 그 이유는 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어지기 떄문이다. -> item.28

그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입(실체화 불가 타입)이 포함되면 알기 어려운 컴파일 오류가 발생한다.

```java
public void print(List<String> ... stringList){ 
  //TODO
}
```
위의 메서드를 생성하면 `List<String>[]` 배열이 생성된다.

## 비 구체화 타입을 가변인자로 사용하면 warning 이 뜬다.

배열은 구체화 타입, 제네릭의 경우 비 구체화 타입이다.

>비 구체화 타입?
> 
> 컴파일 시보다 런타임 시에 더 적은 정보를 갖는 것

## 가변인자에는 제네릭 쓰지마? 

```java
@SafeVarages
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
        }
```

`Arrays.asList(T... a)`, `Collections.addAll(Collection<? super T> c, T... elements)`, E`numSet.of(E first, E... reset)` , ... 메서드가 typesafe 한 경우에 한해서 잘 쓰면 유용한데, 잘못쓰면 ClassCastException을 발생시킬 수 있다.

### 가변인자로 제네릭을 사용하고 있지만 type-safe 한 메서드란?

1. 가변인자를 사용함으로써 만들어지는 배열에 어떠한 것도 덮어쓰지 않는다.
2. 그 배열을 가리키는 참조변수를 직간접적으로 만들지 않는다.

즉, 가변인자가 여러 인자들을 넘겨오는데에만 사용되는 메서드이다.


