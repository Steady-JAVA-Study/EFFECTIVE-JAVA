# ITEM 15. 클래스와 멤버의 접근 권한을 최소화하라

어설프게 설계된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이는 바로 **클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐다.**

잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.

오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다.

정보 은닉, 캡슐화 라고 하는 이 개념은 **소프트웨어 설계의 근간이 되는 원리다.**

## 한 클래스에서만 사용하는 package-private

톱 레벨에 위치한다는 건 같은 패키지의 모든 클래스가 접근할 수 있다는 의미이다. 이 때 private static을 중첩시키면 바깥 클래스 하나에서만 접근할 수 있다.

```java
// AS-IS
public class A{
    private int a;
}
public class B{ // B가 A에서만 쓰이는 클래스라면?
    private int b;
}
```

```java
// TO-DO
public class A{
    private int a;
    
    private static class B{
        private int b;
    }
}
```

## 객체 모델링 방법

```java
public class Post {
    private final Long seq;               // PK(불변)
    private final Id<User, Long> userId;  // USER(불변)
    private final String contents;        // 제목(변함)
    private final int likes;              // 좋아요 갯수(변함)
    private final boolean likesOfMe;      // 좋아요 여부(변함)
    private final int comments;           // 댓글 수(변함)
    private final Writer writer;          // 글쓴이(불변)
    private final LocalDateTime createAt; // 작성일자(불변)
```

1. 초기에는 객체 모델을 수정이 불가능하도록 immutable 상태로 시작 
2. 객체 모델에 필요한 행위가 무엇인지 고민하고, 점진적으로 immutable 상태를 제거
3. (Item 10~12) Object의 equals, hashcode, toString 등의 메서드 구현은 필요에 따라 꼼꼼하게 할 것
4. (Item 11) equals가 참이라면 hashcode의 결과도 참이어야 함
5. 불필요한 생성자는 노출 X
6. (Item 2) 생성 인자 갯수가 많아질 경우 -> 빌더 패턴 고려
7. 꼭 필요한 상수라면 예외적으로 public static final로 공개

```java
public class Post {
  private final Long seq;
  private final Id<User, Long> userId;
  private String contents;
  private int likes;
  private boolean likesOfMe;
  private int comments;
  private final Writer writer;
  private final LocalDateTime createAt;
  
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Post post = (Post) o;
    return Objects.equals(seq, post.seq);
  }

  @Override
  public int hashCode() {
    return Objects.hash(seq);
  }

  @Override
  public String toString() {
    return new ToStringBuilder(this, ToStringStyle.SHORT_PREFIX_STYLE)
      .append("seq", seq)
      .append("userId", userId)
      .append("contents", contents)
      .append("likes", likes)
      .append("likesOfMe", likesOfMe)
      .append("comments", comments)
      .append("writer", writer)
      .append("createAt", createAt)
      .toString();
  }
```

--- 

## 번외

`public static final Thing[] VALUES = {...}` final이지만 배열은 주소를 참조하므로 변경이 가능해진다.

## 첫번째 해결책, 배열을 private으로 만들고, 불변 리스트를 추가

```java
private static fianl PAGE[] PAGE_INFO = {...};
public static final List<PAGE> VALUES = Collections.unmodifiableList(Arrays.asList(PAGE_INFO));

```

## 두번째 해결책, 배열을 private두고, 복사본을 반환하는 public method

```java
private static fianl PAGE[] PAGE_INFO = {...};
public static final PAGE[] values() {
    return PAGE_INFO.clone()
}
```