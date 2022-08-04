# ITEM 40. @Override 애너테이션을 일관되게 사용하라

## @Override

자바에서 기본적으로 제공하는 애너테이션으로 상위 타입의 메서드를 재정의했음을 의미한다.

```java
public class Bigram {

  private final char first;
  private final char second;

  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }

  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
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
    
    System.out.println(s.size());
  }
}
```

위의 코드를 수행하면 Set이므로 26을 기대하지만 결과는 260을 출력한다.

위의 `equals` 의 인자가 Object 타입이 아닌 Bigram이므로 오버라이딩 하지않고 Object 클래스의 equals를 오버로딩하고 있기 때문에 같은 객체로 인식하지 못한다.

```java
    @Override
    public boolean equals(Bigram o) {
        return b.first == first && b.second == second;
    }
```

만약 위와 같이 `@Override`를 추가했으면 컴파일 시점에 오류를 잡을 수 있었다.

즉, 위와 같은 실수를 방지하기 위해서는 항상 `@Override`를 다는 것이 좋다.

