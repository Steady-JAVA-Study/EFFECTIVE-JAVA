# 매개변수가 유효한지 검사하라
- 메서드와 생성자 대부분은 입력 매개변수 값이 특정 조건을 만족하길 want
- 이런 제약은 반드시 문서화하고 메서드 몸체가 시작되기 전에 검사 필요


## 매개변수 검사를 제대로 하지 못했을 시 문제점
- **실패 원자성(failure atomicity)을 어기는 결과 초래 : **
메소드 수행 중간에 모호한 예외를 던지며 실패하거나 잘 수행되더라도 잘못된 결과를 반환 가능
- 메서드가 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 문제가 발생
- 메서드가 잘 수행되지만 잘못된 결과 반환 


> (+) **실패 원자성 : **
호출된 메서드가 실패하더라도 해당 객체는 호출 전의 상태를 유지해야한다.

## 메서드의 입력 매개변수
- 메서드는 입력 매개변수의 값이 특정 조건을 만족하길 원함 
  - 인덱스 값은 음수 X
  - 객체 참조는 null X
- 이러한 제약들은 반드시 문서화 필요 
- 메서드 몸체가 시작되기 전에 검사 필요
- 입력 매개변수가 적절한 값이 들어왔는지 검사 필요
- 매개변수 검사를 하지 않으면 메서드가 예상치 못한 동작을 하거나 잘못된 결과를 반환 가능

##  예외 문서화
- public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화 해야 한다.
- 매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술
	- @throws 자바독 태그를 통해 문서화 가능
- 보통 다음 예외중 하나를 문서화
  - IllegalArgumentException
  - IndexOutOfBoundsException
  - NullPointerException


```java
/**
 * Returns a BigInteger whose value is {@code (this mod m}).  This method
 * differs from {@code remainder} in that it always returns a
 * <i>non-negative</i> BigInteger.
 *
 * @param  m the modulus.
 * @return {@code this mod m}
 * @throws ArithmeticException {@code m} &le; 0
 * @see    #remainder
 */
public BigInteger mod(BigInteger m) {
    if (m.signum <= 0)
        throw new ArithmeticException("BigInteger: modulus not positive");

    BigInteger result = this.remainder(m);
    return (result.signum >= 0 ? result : result.add(m));
}
```

- 위 메서드는 m이 null이면 NullPointerException 을 던지는데도 불구하고 메서드 설명에 그런 말 X
	- 그 이유는 이 설명을 BigInteger 클래스 수준에서 기술했기 때문이다.
- **클래스 수준 주석**을 통해 그 클래스의 모든 public 메서드에 적용할 수 있어서 각 메서드에 일일이 기술하는 것보다 더 깔끔 ~~ 

## 매개변수의 유효성 검사
### 1) java.util.Objects.requireNonNull
```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

- 자바 7에 추가된 java.util.Objects.requireNonNull 메서드를 이용하면 편하게 null 검사를 수동으로 하지 않아도 O 
- 원하는 예외 메시지도 지정 가능
- 특정한 객체 참조가 널인지 판단
- null인 경우 사용자 지정 NullPointException을 발생
	- 이 메소드는 주로 메서드나 생성자의 매개변수 유효성 검증을 위해 설계 
- 생성자나 메소드에서 절대 null이 되면 안되는 매개변수에는 Objects.requireNonNull 사용 가능 

> **동작원리**  
- 동작원리는 아무 타입의 매개변수를 받아 해당 매개변수가 null이면 NPE 발생 
- or 매개변수를 그대로 반환


**사용 예시**
```java

public class Java {

    private Integer count;

    public Java(){

    }

    public Java(Integer i){
        this.count = Objects.requireNonNull(i);
    }
}

```
    

### 2) assert
- 개발/테스트 단계에서 파라미터가 제대로 넘어왔는지 계산이 제대로 됬는지 혹은 특정 메소드가 작동하는 한계상황(Null이 들어오면 작동안함)을 정하는 용도로 사용 

```java

class AssertionTest {  
 public static void main( String args[] ){  
      
  int value = -1;  
  assert value>=0:" 음수값입니다.";  
  
  System.out.println("값 : "+ value);  
 }   
}
```

```
Exception in thread "main" java.lang.AssertionError: 음수값입니다.
```

1) expression1의 조건이 '참'이면 아래 구문을 실행하고 & '거짓'이면 AssertionError 예외가 발생
```
assert expression1;
```

2) expression1의 조건이 '참'이면 아래 구문을 실행 &'거짓'이면 AssertionError 예외와 함께 expression2가 예외로 출력 
```

assert expression1: expression2;
```

- 공개되지 않은 메서드라면 패키지 제작자인 사람이 메서드가 호출되는 상황을 통제 가능 

- public이 아닌 메서드라면 단언문(assert)를 사용하여 매개변수 유효성을 검증 가능 
- assert 문은 실패하면 AssertionError를 던짐
- 런타임에 아무런 효과도, 아무런 성능 저하 X


### 3) 나중에 쓰려고 저장하는 매개변수의 유효성을 검사해라
- 직접 사용하지 않으나 나중에 쓰기 위해 저장하는 매개변수는 더욱 신경써서 검사 필요
- 생성자의 유효성 검사는 불변식을 지키기 위해 꼭 필요 


## 메서드 매개변수 유효성 검사에도 예외
- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 수행될 때 

```jaca
public class Collectionsorting 
{ 
    public static void main(String[] args) 
    { 
        // Create a list of strings 
        ArrayList<String> al = new ArrayList<String>(); 
        al.add("Geeks For Geeks"); 
        al.add("Friends"); 
        al.add("Dear"); 
        al.add("Is"); 
        al.add("Superb"); 
  
        /* Collections.sort method is sorting the 
        elements of ArrayList in ascending order. */
        Collections.sort(al); 
  
        // Let us print the sorted list 
        System.out.println("List after the use of" + 
                           " Collection.sort() :\\n" + al); 
    } 
}
```
- Collections.sort(List) 처럼 정렬하는 메서드는 리스트 안의 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정에서 이 비교가 이뤄짐

- 상호 비교될 수 없는 타입의 객체가 들어 있다면, 그 객체와 비교할 때 ClassCastException 던짐 

- 따라서 비교하기 앞서 리스트 안의 모든 객체가 상호 비교될 수 있는지 검사해봐야 이득 X

## 결론
- 이번 아이템은 매개변수에 제약을 두는 게 좋다라고 해석하지 말아야 한다.
- 메서드는 최대한 범용적으로 설계해야 하고, 동작을 제대로 한다면 제약은 적을수록 좋다.

_____________________________________________________

- 실제적으로 발생할 수 있는 오류에 대해서 인지하고 미리 예외를 만들 수 있도록 한다. 이 예외에 대한 주석, 설명은 필수
- 무조건적으로 빠르게 검사하는 것이 아닌 암묵적인 검사의 상황과, 때로는 예상되지 않는 실패의 경우 던지는 예외에 대해서도 고려
- 매개변수에 집착해서 제약을 두는 것이 아니라 범용적으로 설계하기

## reference
https://greeng00se.tistory.com/m/66
https://codingwell.tistory.com/131
https://emgc.tistory.com/124