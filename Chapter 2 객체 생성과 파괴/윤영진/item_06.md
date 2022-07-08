# ITEM 6. 불필요한 객체 생성을 피하라

## 객체를 매번 생성하기보다는 재사용하는 편이 좋다.

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다. 

```java

String s = new String("bikini"); // 따라 하지 말 것 
        
for (int i = 0; i < 100000000; i++) {
        String str = new String("java");
}
```

생성자를 통해 String 객체를 만들게 되면 쓸데없이 String 인스턴스를 반복해서 만들게 된다.

실행될 때마다 Strng 인스턴스를 새로 만든다. 인스턴스를 새로 만들때마다 `heap` 영역에 `String` 인스턴스가 저장이 된다.

```java

String s = "bikini";

```

위의 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다. 문자열 리터럴을 통해서 `heap` 영역의 `String Constant Pool`에 저장되어 인스턴스가 재사용 된다. 

Java의 가상머신은 똑같은 문자열 리터럴에 대해서는 동일 코드를 사용하는 재사용성이 보장된다.


![image](https://user-images.githubusercontent.com/83503188/177571054-a84190fa-3dd8-45b2-be04-66d08f86331f.png)

## 정적 팩터리 메서드를 이용하여 불필요한 객체 생성을 방지하자

생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.

자바에서 사용되는 Boolean 객체를 통해 어떻게 불필요한 객체 생성을 방지하는지 살펴보자.

```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {

    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
    
    // ...
    public static boolean parseBoolean(String s) {
        return "true".equalsIgnoreCase(s);
    }

    // ...
    public static Boolean valueOf(String s) {
        return parseBoolean(s) ? TRUE : FALSE;
    }
}
```

코드를 보면 Boolean.valueOf(String)정적 팩터리 메서드를 사용하여 미리 생성된 `TRUE`, `FALSE`를 반환하는 것을 확인할 수 있다.

## 객체 생성이 비싼 경우 캐싱을 통해 객체 생성을 방지해보자.

생성 비용이 아주 비싼 객체도 있다. 이런 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하길 권한다.

비싼 객체란 말은 인스턴스를 생성하는데 드는 비용이 크다는 의미이다. 즉, 메모리, 디스크 사용량, 대역폭 등이 높을수록 생성 비용이 비싸다고 한다.

```java
static boolean isRomanNumeral(String str) {
    return str.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

`String.matches`는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다.

그 이유는 내부적으로 Pattern 인스턴스를 만들어 사용하는데 Pattern 인스턴스는 입력받은 정규표현식의 유한 상태 머신을 만들기 때문에 생성 비용이 높다. 이런 생성 비용이 높은 Pattern 인스턴스를 한 번 쓰고 버리는 구조로 만들어 곧바로 GC의 대상이 되게 만들고 있다. 즉, 비싼 객체라고 할 수 있다.

> 유한 상태 머신
> 
> https://www.youtube.com/watch?v=ZfW7FwuBd90

이런 문제를 개선하기 위해서는 `Pattern` 객체를 만들어 컴파일하고 재사용하는 것이 좋다.

```java
public class RomanNumber {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String str) {
        return ROMAN.matcher(str).matches();
	}
}
```

이렇게 개선하면 inRomanNumeral이 빈번히 호출되는 상황에서 성능을 상당히 끌어올릴 수 있다.

개선된 isRomanNumeral 방식의 클래스가 초기화된 후 이 메서드를 한 번도 호출하지 않는다면 ROMAN 필드는 쓸데없이 초기화된 꼴이다. isRomanNumeral 메서드가 처음 호출될 때 필드를 초기화하는 지연 초기화(lazy initialization, 아이템 83)로 불필요한 초기화를 없앨 수는 있지만, 권하지는 않는다. 

지연 초가화는 코드를 복잡하게 만드는데, 성능은 크게 개선되지 않을 때가 많기 때문이다.

## 오토박싱을 사용할 때 주의하라.

오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.

하지만 이를 잘못 사용하게 되면 불필요한 메모리 할당과 재할당을 반복하여 성능이 느려질 수 있다.

```java
void autoBoxing_Test() {
        Long sum = 0L;
        for(long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
        }
        }

        void noneBoxing_Test() {
        long sum = 0L;
        for(long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i;
        }
        }
```
![image](https://user-images.githubusercontent.com/83503188/177574585-2a76e0af-2486-4b9d-9525-36a310f3f43c.png)

`Long` 타입에서 `long` 타입으로 변경해주는 것 만으로도 엄청난 성능 차이가 나는 것을 확인할 수 있다.

**박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**

## 주의

이번 아이템의 내용을 **객체 생성은 비싸니 무조건 피해야한다로 오해하면 안된다.**

최근 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일은 크게 부담되지 않는다. 프로그램의 명확성, 간결성, 기능을 위해서 객체를 생성하는 것이라면 일반적으로 더 권장되는 일이다.

다만, 정말 아주 무거운 객체가 아닌 이상, 단순히 객체 생성을 피하고자 본인만의 객체 풀을 만들지 말자. JVM의 가비지 컬렉터는 상당히 잘 최적화 되어있어 가벼운 객체를 다룰 때는 본인이 만든 객체 풀보다 훨씬 빠르다.

객체 풀을 만드는게 나은 예가 있긴 하다. 데이터베이스 연결 같은 경우 생성 비용이 워낙 비싸니 재사용하는 편이 낫다. 하지만 일반적으로는 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨린다.

> 객체 풀(Object pool)이란?
> 
> 객체를 매번 할당, 해제하지 않고 고정 크기 풀에 들어 있는 객체를 재사용함으로써 메모리 사용 성능을 개선한다. 메모리 단편화를 방지한다. 게임이 실행될 때 메모리를 미리 크게 잡아놓고 쓴다. 