# 적시에 방어적 복사본을 만들라
 
- 자바는 메모리 충돌 오류에서 안전한 언어
- 자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식이 지켜진다. 
- BUT 다른 클래스로부터의 침범을 아무런 노력없이 다 막을 수 있는건 아니므로 방어적으로 프로그래밍해야 함
- 어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능, BUT 실수로 내부를 수정하도록 허락하는 경우가 생긴다.

## 불변식 지키지 못한 클래스
```java
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) # 이 조건 늘 성립해야 함 
            throw new IllegalArgumentException( );
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }

        public static void main(String args[]){
                Date start = new Date();
                Date end = new Date();
                Period p = new Period(start, end);
                end.setYear(78); // Modifies internals of p!
        }
 
}
```
- 원래 main 에서 start 가 end 보다 항상 더 빠름
- 그러나 Date 는 불변이 아님, 
- 위 Date가 가변이라는 사실을 이용하면 어렵지 않게 이 클래스의 불변식을 깨뜨리기 가능 
- Date 가 가변이라는 사실 때문에 그 불변식을 깨뜨리기 가능 

## 외부로부터 내부 보호하는 법
1) 외부에서 받은 매개변수를 복사해서 저장 
2) 내부에서 외부로 데이터를 전달할 때 복사해서 반환 

## 방어적 복사(defensive copy)
- 
- 외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy) 
- Period 인스턴스 안에서 원본이 아닌 복사본을 사용
- 반드시 방어적 복사본을 만들고 이 복사본으로 유효성을 검사
`방어적 복사본 생성한 Period.java` 

```java
// 수정한 생성자(직접 대입이 아니라 복사)
   public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); # 복사 
        this.end = new Date(end.getTime()); # 복사 
# 반드시 방어적 복사본을 만든 후에,  이 복사본으로 유효성을 검사 !!! 
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException();
    }

  //  public Period(Date start, Date end) {
  //      if (start.compareTo(end) > 0) # 이 조건 늘 성립해야 함 
  //          throw new IllegalArgumentException( );
  //      this.start = start; # 이전에는 직접 대입 
  //      this.end   = end;
  //  }
# 접근자가 가변 필드의 방어적 복사본을 반환하게 하면 된다.
# 필드의 방어적 복사본을 반환 
    public Date start() {
        return new Date(start.getTime());
    }
# 필드의 방어적 복사본을 반환 
    public Date end() {
        return new Date(end.getTime());
    }
    
 //   public Date start() {  기존 코드 
 //        return start;
 //   }
 //   public Date end() {
 //       return end;
 //   }

        public static void main(String args[]){
                Date start = new Date();
                Date end = new Date();
                Period p = new Period(start, end);
                end.setYear(78); // Modifies internals of p!
        }
 
}
```
- 매개변수의 유효성 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사
- 멀티스레드 환경에 대응하기 위해서
- 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다. 방어적 복사를 매개변수 유효성 검사 전에 수행하면 이런 위험에서 해방 가능 
- 클라이언트가 반환된 Date를 아무리 수정해도 내부의 Date는 변경 x ! 

### clone 메서드 사용하지 않기

- 위 예시에서는 Date의 하위 클래스가 clone() 메서드를 재정의할 수 있기 때문에 안전하지 않다.
- 접근자에서는 사용 가능 

> 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들때 clone을 사용해서 X

- 방어적 복사에 Date의 clone 메서드를 사용 X
- Date는 final 이 아니므로 clone이 Date가 정의한게 아닐 수도 있음
- SO, clone이 악의를 가진 하위 클래스의 인스턴스를 반환 가능 

 

- 위 코드에서 매개변수 start, end 가 확장된 Date의 하위 클래스일 수도 있다. 그래서 clone()을 호출했을때 Date 클래스의 clone 메서드가 아닌 확장된 **하위 클래스의 clone 메서드를 호출할 수도 있음** 

- 이 하위 클래스는 start, end 필드의 참조를 pricate 정적 리스트에 담아뒀다가 공격자에게 이 리스트에 접근하는 길을 열어줄 수도 있다 

→ 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들때 clone을 사용해서 X

## 결론 
- 되도록 불변 객체들을 조합해서 객체를 구성 

- 그것이 안되면 방어적 복사를 한다.

- 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략해도 된다. (예를 들어 같은 패키지의 경우)이런 경우에도 문서화하는 것이 좋다.

- 특히 통제권을 넘겨주거나 생성자를 가진 클래스들은 이러한 공격에 취약

## reference
https://wjdtn7823.tistory.com/90
https://daebalpri.me/entry/Effective-Java-3E-ITEM-50-%EC%A0%81%EC%8B%9C%EC%97%90-%EB%B0%A9%EC%96%B4%EC%A0%81-%EB%B3%B5%EC%82%AC%EB%B3%B8%EC%9D%84-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%9D%BC
