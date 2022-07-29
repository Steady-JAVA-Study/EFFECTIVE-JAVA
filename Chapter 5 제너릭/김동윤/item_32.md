# 제네릭과 가변인수를 함께 쓸 때는 신중하라

## 가변인자 복습
```java
public static void display(String ...infos) {
    for (String info: infos) {
        System.out.println(s);
    }
}
```

```java
public class test {
	public static void main(String[] args) {
		test t = new test();
		t.variable();
		t.variable("A");
		t.variable("A","B");
		t.variable("A","B","C");
		t.variable("A","B","C","D");
	}
	
	public void variable(String... s) {
		System.out.println(s);
	}
}
```
> 키워드 `...` 을 사용
- 가변인자는 컴파일시 **배열**로 처리
- 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절 가능케 해줌 
- 이 가변인수들을 담기 위한 배열이 자동으로 만들어짐 =>이 배열이 클라이언트에 노출


________________________________________________________

## 제네릭타입의 가변인수 메서드를 호출하면 어케 되냐?

```java
    public static void main(String[] args) {
        List<String>[] stringLists = new List<String>[1]; // (1)
        List<Integer> intList = List.of(42); // (2)
        Object[] objects = stringLists; // (3)
        objects[0] = intList; // (4)
        String s = stringLists.get(0); // (5) 컴파일 에러가 난다!
    }
}
```

```
(1)이 허용된다고 가정

(2)는 원소가 하나인 List<Integer>를 생성

(3)은 (1)에서 생성한 List<String>의 배열을 Object 배열에 할당
이때, 배열은 공변이므로 아무런 문제x

(4)는 (2)에서 생성한 List<Integer>의 인스턴스를 
Object 배열의 첫 원소로 저장, 제네릭은 소거 방식(runtime , remove type) 으로 구현 => 성공

즉, 런타임에는 List<Integer> 인스턴스의 타입은 
단순히 List가 되고, 
List<Integer>[] 인스턴스의 타입은 List[]

따라서 (4)에서도 ArrayStoreException X

List<String> 인스턴스만 담겠다고 선언한 
stringLists 배열에는 현재 List<Integer> 인스턴스가 저장 (compile check x)

컴파일러는 꺼낸 원소를 자동으로 String으로 형변환하는데,
이때 원소의 타입이 Integer와 String으로 다르다. 

=> 따라서 런타임에 ClassCastException이 발생한다.

```
## so 제네릭 배열이 생성되지 않도록 해야 한다

## 그럼에도 불구하고 varargs 매개변수를 받는 메서드를 선언할 수 있게 한 이유?
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용
- 자바 7부터는 @SafeVarargs 를 이용해 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 oo
- @SafeVarargs은 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 달면 된다.
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 안전한지 확인하는 방법은 **varargs 매개변수 배열이 인수들을 전달하는 일만 한다면** 그 메서드는 안전하다


## @SafeVarargs 언제 사용해 ?
다음 두 조건을 모두 만족하는 제네릭 varargs 메서드는 안전

1. varargs 매개변수 배열에 아무것도 저장 x (덮어쓰지않고)

2. 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 X , that is, 신뢰할 수 없는 코드가 배열에 접근할 수 없다면!!

## 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 X examples 

```java
public class Main2 {
    public static void main(String[] args) {
    
    // pickTwo의 반환값을 attributes 에 저장하기 위해 
    //String[] 로 형변환하는 코드를 컴파일러가 자동 생성
    
        String[] attributes = pickTwo("AA", "BB", "CC");
    }
    
    //pickTwo return  타입은 Object[]
    //Object[] 는 String[]의 하위타입이 아니므로 이 형변환은 실패
 
    static <T> T[] toArray(T...args) { // var args 
        return args;
    }
 
    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
         //toArray 에 넘길 T 인스턴스 2개를 담을 
         // varargs 매개변수 배열을 만드는 코드를 생성 & 
         // 해당 배열의 타입은 Object[]
            case 0 : return toArray(a,b); //=> 타입은 Object[]
            case 1 : return toArray(a,c);
            case 2 : return toArray(b,c);
        }
 
        throw new AssertionError(); 
    }
}
```

1) `TO ARRAY `: var args  USE
2) `PICK TWO` CALLS `TO ARRAY` 
3) `PICK TWO` RETURNS Object[] 타입 배열
4)  main ->  `String[] attributes = pickTwo("AA", "BB", "CC");`
String <- Object nono, however we can check this in Runtime , cuz 
배열은 실체화, 배열은 런타임에 자신이 담기로 한 원소의 타입을 인지하고 확인,

=> RUNTIME ERROR 
SO 네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 X 


## 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 점 2가지 예외

1) @SafeVarargs 로 제대로 어노테이션된 또다른 varargs 메서드에 넘기는 것은 안전 (CONFIRMED)

2) 그저 이 배열 내용의 일부 함수를 **호출만**하는(**varargs 를 받지않는**) 일반 메서드에 넘기는 것도 안전하다.

```java
@SafeVarargs //annotation essential

				// 임의 개수 리스트를 인수
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
 
 	// 받은 순서대로 그 안의 모든 원소를 
    // 하나의 리스트로 옮겨 담아 반환
    for (List<? extends T> list : lists) {
    result.addAll(list);
    }
 
    return result;
}
```

- @SafeVarargs 어노테이션을 사용할때는 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달gi
- 그래야 사용자를 헷갈리게 하는 컴파일러 경고를 없앨 수 oo
<=>  안전하지 않은 varargs 메서드는 절대 작성 no no ~ 

## 결론
- 가변인수와 제네릭은 낫 굿 ~~ 호환성
**이유**
1. 가변인수 기능은 배열을 노출하여 추상화가 완벽 X
2. 배열과 제네릭의 타입 규칙이 서로 다르기 때문

- 제네릭 varargs 매개변수는 타입 안전하지는 않지만, 허용된다.
- 메서드에 제네릭 (혹은 매개변수화된) varags 매개변수를 사용하고자 한다면, 
먼저 그 메서드가 타입 안전한지 확인 &  @SafeVarargs 애너테이션을 달아 사용하는 데 불편함이 없게 하자

&& rule
1) varargs 매개변수 배열에 아무것도 저장 NO
2) 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출 NO 

## 출처 
https://incheol-jung.gitbook.io/docs/study/effective-java/undefined-3/2020-03-20-effective-32item
