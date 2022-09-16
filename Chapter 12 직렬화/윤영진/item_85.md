# ITEM 85. 자바 직렬화의 대안을 찾으라

> 직렬화?
> 
> `java.io.Serializable` 인터페이스를 상속받은 객체를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하는 기술
> 
> 역직렬화는 반대로 byte로 변환된 data를 Object로 변환하는 기술

```java
public class Member implements Serializable {
    private String name;
    private String email;
    private int age;

    public Member(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }
    @Override
    public String toString() {
        return String.format("Member{name='%s', email='%s', age='%s'}", name, email, age);
    }
}
```

```java
public static void main(String[] args){
    Member member = new Member("홍길동", "hong@gmail.com", 25);
    byte[] serializedMember;
    try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
        try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
            oos.writeObject(member);
            serializedMember = baos.toByteArray();
        }
    }
    System.out.println(Base64.getEncoder().encodeToString(serializedMember));
}
```

## 문제점

- 1997년, 자바에 처음으로 직렬화가 도입
- 직렬화는 프로그래머가 어렵지 않게 분산 객체를 만들 수 있다는 구호는 매력적이었지만, 보이지 않는 생성자, API와 구현 사이의 모호해진 경계, 잠재적인 정확성 문제, 성능, 보안, 유지보수성 등 그 대가가 크다
- 직렬화의 근본적인 문제는 공격 범위가 너무 넓고 지속적으로 더 넓어져 방어하기 어렵다는 점이다.

## 대안

### JSON
- JSON은 더글라스 크록퍼드가 브라우저와 서버의 통신용으로 설계
- 언어 중립적이지만 자바 스크립트용으로 만들어진 흔적이 있음

### 프로토콜 버퍼

- 구글이 서버 사이에 데이터를 교환하고 저장하기 위해 설계
- 프로토콜 버퍼는 C++용으로 만들어졌고 아직 그 흔적이 남아있음

| JSON                                                         |                 Protocol Buff                  |
|--------------------------------------------------------------|:----------------------------------------------:|
| JavaScript Object Notation 경량 데이터 교환 형식                      | 데이터를 인코딩하는 방법 Google에서 구조화된 데이터 직렬/역직렬화를 위해 개발 |         
| No schema                                                    |            메세지를 정의하고, 교환하기 위한 규칙 필요            |  
| Text 형식(key, value쌍)                                         |                     Binary                     | 
| 가볍고 다른 직렬화 기술들보다 빠르다                                         |                   JSON보다 빠르다                   | 
| 문자열, 숫자, JSON객체, 배열, boolean, null (class와 function 지원하지 않음) |                 폭넓은 데이터 유형 지원                  | 


