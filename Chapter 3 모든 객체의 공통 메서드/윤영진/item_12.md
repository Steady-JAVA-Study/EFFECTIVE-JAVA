# ITEM 12. toString을 항상 재정의하라

Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다. 이 메서드는 `PhoneNumber@adbbd`처럼 단순히 `클래스_이름@16진수로_표시한_해시코드`를 반환할 뿐이다.

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

## toString() 사용

1. 콘솔로 객체를 확인할 때 
   1. System.out.println()
   2. System.out.print()
   3. 객체에 문자열을 연결(SomeObject + "")
   4. assert 구문 사용 시
   5. 등등...
2. 디버깅 할 때
3. 로깅할 때

## toString()을 왜 항상 재정의해야할까요?

사용하는 예시로 콘솔로 확인하거나, 디버깅 하거나, 로깅할 때를 들었습니다. 예시로 든 모든 상황은 사용하는 주체가 개발자다.

즉, 개발자에게 객체가 보여지는데 클래스_이름@16진수로_표시한_해시코드 와 같은 방식으로 보여진다면 해당 객체 안에 뭐가 들었는지 알 수 없다.

사용하려는 객체에 toString()을 재정의하면 로깅 및 디버깅 시 개발자에게 좀 더 유익한 정보를 전해줄 수 있다.

`{Jenny=PhoneNumber@abbd}` -> `{Jenny=707-867-5309}`

## 어떻게 재정의해야 하는가?

### 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 담아야 한다.

```text
PhoneNumber@adbbd -> 012-1234-5678   
Car@442           -> Car{name=sun, position=2}
```
### 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.

```text
class Address {
    private final String city;
    private final String gu;
    private final String dong;
    private final String detail;

    Address(String city, String gu, String dong, String detail) {
        this.city = city;
        this.gu = gu;
        this.dong = dong;
        this.detail = detail;
    }

    @Override
    public String toString() {
        return "Address{" +
                "city='" + city + '\'' +
                ", gu='" + gu + '\'' +
                '}';
    }
}
```

위의 코드처럼 일부만 반환하는 것은 좋은 방법이 아니다.
- 만약 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 요약정보를 담아야 한다. `ex) 맨해튼 거주자 전화번호부(총 1487536개)`

### toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다.

전화번호부나 행렬 같은 값 클래스라면 문서화를 권장한다. 포맷을 명시하면 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.

#### 포맷 명시 -> 포맷을 명시하든 아니든 의도는 명확히 밝혀야 한다.
```java
 /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
    @Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
```

```text
// 포맷 적용 전,
PhoneNumber{areaCode='02', prefix='512', lineNumber='1234'}

// 포맷 적용 후,
02-512-1234
```

다만 포맷 명시에도 단점이 있다. 포맷을 한번 명시하면 (그 클래스가 많이 쓰인다면) 평생 그 포맷에 얽매이게 된다.

반대로 포맷을 명시하지 않는다면 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 얻게 된다. 

