# 아이템1 생성자 대신 정적 팩터리 메서드(static factory method)를 고려하라

> 직접적으로 생성자를 통해 객체를 생성하는 것이 아닌 메서드를 통해서 객체를 생성하는 것

- public static 메서드를 통해 해당 클래스의 인스턴스 얻기 가능 

## 1. 이름을 지을 수 있어서 어떤 매개변수로 인스턴스를 생성하는지 알기 쉬움 

(1) Car 인스턴스 생성 시 브랜드로 생성함을 알 수 있는 코드 
```java
public class Application {
    public static void main(String[] args) {
        Car car = Car.brandOf("현대");
    }
}

class Car {
    private String brand;

    private Car(String brand) {
        this.brand = brand;
    }
    
    public static Car brandOf(String brand) {
        return new Car(brand);
    }
}

```

(2)
- 같은 로또 인스턴스를 반환하더라도 똑같은 로또가 아니고 자동로또인지 수동로또인지 분간할 수 있게 해줌 
```java

public class LottoFactory() {
  private static final int LOTTO_SIZE = 6;

  private static List<LottoNumber> allLottoNumbers = ...; // 1~45까지의 로또 넘버

  public static Lotto createAutoLotto() {
    Collections.shuffle(allLottoNumbers);
    return new Lotto(allLottoNumbers.stream()
            .limit(LOTTO_SIZE)
            .collect(Collectors.toList()));
  }

  public static Lotto createManualLotto(List<LottoNumber> lottoNumbers) {
    return new Lotto(lottoNumbers);
  }
  ...
}

```
(3) 어떤 매개변수로 책 생성하는지 알기 easy
```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    // 생성자를 이용한 객체 초기화
    public Book(String name) {
        this.name = name;
    }

    // 펙토리 메서드를 이용한 객체 초기화
    public static createBookWithName(name) {
        return new Book(name);
    }
}
```
클라이언트 호출 코드

```java
Book book1 = new Book("effeciveJava"); 
// "effectiveJava" 가 어떤 인스턴스 변수인지 알기 어렵다.
        
Book book2 = Book.createBookWithName("effectiveJava"); 
// 이름이 "effectiveJava" 임을 한눈에 알 수 있다.


```

## 2. 호출 할 때마다 인스턴스를 새로 생성하지 않아도 OK
정적 팩터리 메서드를 통해 불변(immutable) 클래스 는 
① **인스턴스를 만들어놓거나** 
② 새로 생성한 **인스턴스를 캐싱하여 재활용** 하는 식으로 
불필요한 객체생성을 피할 수 있다

- 정적 팩터리 메서드는 인스턴스 통제도 한다.
=> 언제 어떤 인스턴스 살아있게 할 지 통제하는 것 
- 인스턴스를 통제하는 이유
   - 싱글턴을 만들기 위해
   - 인스턴스화 불가로 만들기 위해
   - 불변 클래스에서 값(value)이 같은 인스턴스는 단 하나임을 보장

## 3. 반환 타입의 하위 타입 객체를 반환
- 공통 로직 수행에도 유용~!
- Payment 라는 상위타입 Interface 로 하위 타입인 SamSungPayment 클래스를 숨김으로써
- 프로그래머는 `인터페이스대로 동작할 객체를 얻을 것을 알기에`  굳이 SamSungPayment.class 를 찾아가서 알아보지 않아도된다.
> 사용자들이 해당 인터페이스의 구현체를 일일이 알아 볼 필요가 없다

- 구체적인 클래스들 SamSungPayment , KakaoPayment 이 반환된다 하지만 클라이언트측에서 반환받는 객체는 Payment 인터페이스이기 때문에 구체적인 클래스를 몰라도 얼마든지 사용하는데 문제가 없다

```java
static class CARD{
    public static final String SAMSUNG_CARD = "삼성";
    public static final String KAKAO_CARD="카카오";

    static Payment payment(String card){
        switch (card){
            case SAMSUNG_CARD:
                // 3. 반환 타입의 하위타입 객체를 반환할 수 있는 능력이 있다.
                return new SamSungPayment();
            case KAKAO_CARD:
                return new KakaoPayment();
        }
        throw new IllegalArgumentException();
    }

}
```

## 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환
(ex) EnumSet 클래스
> EnumSet 의 요소가 64 개 이하면 ReqularEnumSet 의 인스턴스 return
EnumSet 의 요소가 65 개 이상이면 JumboEnumSet 의 인스턴스 return

- 클라이언트 측에서는 구체적인 클래스를 몰라도 사용하는데 아무 문제가 발생X

_______________

| **명명 규칙**                  | **설명**                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| **from()**                     | 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메소드.                                                                                                 ex) Date.from() |
| **of()**                       | 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메소드.                                                                                                    ex) Enum.of() |
| **valueOf()**                  | from 과 of 의 더 자세한 버전                                                                                                                                                                            ex) BigInteger.valueOf() |
| **instance()  getInstance()**  | (매개 변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만 같은 인스턴스임을 보장하지는 않는다.                                                        ex)StackWalker.getInstance() |
| **create()** **newInstance()** | instance 혹은 getInstance와 같지만 매번 새로운 인스턴스를 생성해 반환함을 보장한다.                                                                                       ex) Array.newInstance() |
| **getType()**                  | getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다.                                                                             ex)Files.getFileStore() |
| **newType()**                  | newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다.                                                                                 ex)Files.newBufferedReader() |
| **type()**                     | getType과 newType의 간결한 버전                                                                                                                                                                                  ex)Collections.list() |

---

## 출처
이펙티브 자바 도서
https://tecoble.techcourse.co.kr/post/2020-05-26-static-factory-method/
https://7942yongdae.tistory.com/147 
https://github.com/Meet-Coder-Study/book-effective-java