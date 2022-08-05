# @Override 애너테이션을 일관되게 사용하라
- 추상메소드를 구현할 때를 제외하고는 @Override 사용
- `@Override` 의 의미는 상위 클래스의 메서드를 재정의 했음을 의미

## @Override를 선언하지 않은 메서드
```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); //260
    }
}
```
- HashSet은 내부적으로 equals 메서드를 기반으로 객체의 논리적 동치적(equals) 검사를 실시 
- 하지만 자세히 보면 equals메서드의 파라미터 타입이 Bigram
- equals 메서드를 재정의 한게 아니라 Overloading 한 꼴
=> equals를 재정의 하려면 파라미터 타입이 Object

```java
@Override // 어노테이션 붙이면 
public boolean equals(Bigram bigram) {
    return bigram.first == this.first &&
        bigram.second == this.second;
}
``

`컴파일 에러`
```
Error:(15, 5) java: method does not override or implement a method from a supertype
```

=> no method with the same signature (i.e. the parameters) can be found in the supertype, 니가 지금 오버라이딩하는 애의 원천을 찾을 수가 없어~ 

```java
@Override
public boolean equals(Object bigram) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == this.first &&
        b.second == this.second;
}
```

## 결론
- 상위 클래스의 메서드를 재정의 하는 모든 메서드에 @Override 애너테이션을 달자
- 굳이 @Override를 달지 않아도 동작은 한다. 하지만 일괄적으로 붙여주는게 좋다.
- 인터페이스를 상속한 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아있다면,
- 컴파일러가 바로 사실을 알려준다.
- Java 8 부터 Default 메서드의 사용이 가능해 지면서, 인터페이스의 메서드를 재정의 할 때도 사용할 수 있다.
- 구현하려는 인터페이스에 Default 메서드가 없음을 안다면 @Override를 생략해 코드를 조금 깔끔히 유지해도 좋다.
- 웬만하면 추상클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스를 재정의하는 모든 메서드에 @Override를 다는것이 nice 
(실수 했을 때 컴파일러가 알려준다.)

## reference 
https://jaehun2841.github.io/2019/02/04/effective-java-item40/#Override%EB%A5%BC-%EC%84%A0%EC%96%B8%ED%95%98%EC%A7%80-%EC%95%8A%EC%9D%80-%EB%A9%94%EC%84%9C%EB%93%9C