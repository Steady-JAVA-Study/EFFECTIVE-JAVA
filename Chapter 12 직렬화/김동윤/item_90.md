# 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
> Serializable을 구현하기로 결정한 순간부터 생성자 이외의 방법으로 인스턴스를 생성 가능
- 직렬화 프록시 패턴을 사용하면 위험을 크게 줄이기 가능 
   - 안정적으로 역직렬화를 할 수 있는 직렬화 프록시 패
## 직렬화 프록시 패턴
- 이게 뭔데?
> - 객체를 직렬화 하는데 실제 직렬화 되는 클래스를 이용하는 것이 아니라 별도 직렬화를 해주는 프록시 클래스를 둬서 그 클래스를 이용해 직렬화를 수행하는 패턴

### <직렬화 프록시 패턴 적용 이전~>
```java
public class Period implements Serializable {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        
        if(this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }
}
```

- Period에서 end는 언제나 start보다 크거나 같아야 함
- 이를 위해 생성자에서는 end가 start보다 작을 경우 IllegalArgumentException을 발생
- **문제** : 생성자를 통해 생성할 경우 start, end에 대한 불변식을 체크하고 Period를 생성하지만 직렬화된 바이트 스트림을 통해 인스턴스를 역직렬화 해 생성하는 경우에는 생성자에 있는 start, end 값 체크를 하지 않음
- **oh my gosh **~~ so, 공격자가 end가 start보다 작은 Period의 바이트 스트림을 생성해서 보낸다면 받는 쪽에서는 불변식이 깨진 것을 모르고 Period로 역직렬화 해 사용 가능

> - 만일 역직렬화할 바이트 스트림을 보내주는 곳의 출처가 정확하고 역직렬화할 바이트 스트림이 잘못된 값이 아니라는 확신이 있다면 상관 없지만, 그러한 확신이 없고 꼭 직렬화를 해야하는 경우라면 어떻게 해야할까? 
   - 이럴 때 직렬화 프록시 패턴이라는 것을 사용 => _**공격자가 의도한대로 값을 변경 못하게 막기 가능**_

### <직렬화 프록시 패턴 적용 이후~>
> 복습 ㅣ  실제 직렬화 되는 클래스를 이용하는 것이 아니라!! 별도 직렬화를 해주는 프록시 클래스를 둬서 그 클래스를 이용해 직렬화를 수행하는 패턴

```java
public final class Period implements Serializable {

  private final Date start;
  private final Date end;

  public Period(Date start, Date end) {
    if (this.start.compareTo(this.end) > 0) {
      throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    }
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
  }

  //자바의 직렬화 시스템이 바깥 클래스의 인스턴스 말고 SerializationProxy의 인스턴스를 반환하게 하는 역할
  //이 메서드 덕분에 직렬화 시스템은 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다
  //"프록시야 너가 대신 직렬화되라"
  private Object writeReplace() {
    return new SerializationProxy(this);
  }

  //불변식을 훼손하고자 하는 시도를 막을 수 있는 메서드
  //"Period 인스턴스로 역직렬화를 하려고해? 안돼"
  private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다");
  }

  //바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스(Period의 직렬화 프록시)
  private static class SerializationProxy implements Serializable {

    private static final long serialVersionUID = 234098243823485285L;

    private final Date start;
    private final Date end;

    //생성자는 단 하나여야 하고, 바깥 클래스의 인스턴스를 매개변수로 받고 데이터를 복사해야 함
    SerializationProxy(Period p) {
      this.start = p.start;
      this.end = p.end;
    }

    //역직렬화시 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해줌.
    //역직렬화는 불변식을 깨뜨릴 수 있다는 불안함이 있는데, 
    //이 메서드가 불변식을 깨뜨릴 위험이 적은 정상적인 방법(생성자, 정적 팩터리, 다른 메서드를 사용)으로 역직렬화된 인스턴스를 얻게 한다.
    //"역직렬화 하려면 이거로 대신해"
    private Object readResolve() {
      return new Period(start, end);
    }
  }
}
```
- 직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단

- 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단 
- 필드들을 final로 선언해도 되므로 Period 클래스를 진정한 불변으로 만들기 가능
- 역직렬화때 유효성 검사를 하지 X 가능
- 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동

## 결론
제 3자가 확장할 수 없는 클래스라면 직렬화 프록시 패턴을 사용 ~ 

### reference
https://pamyferret.tistory.com/m/58
https://greeng00se.tistory.com/116
https://github.com/Meet-Coder-Study/book-effective-java/blob/main/12%EC%9E%A5/90_%EC%A7%81%EB%A0%AC%ED%99%94%EB%90%9C_%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4_%EB%8C%80%EC%8B%A0_%EC%A7%81%EB%A0%AC%ED%99%94_%ED%94%84%EB%A1%9D%EC%8B%9C_%EC%82%AC%EC%9A%A9%EC%9D%84_%EA%B2%80%ED%86%A0%ED%95%98%EB%9D%BC_%ED%99%A9%EC%A4%80%ED%98%B8.md