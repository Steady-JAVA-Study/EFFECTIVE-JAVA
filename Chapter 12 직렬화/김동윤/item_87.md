# 커스텀 직렬화 형태를 고려해보라
## 기본 직렬화 형태
객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방
> - 물리적 표현 → 코드로 어떻게 구현했는지
- 논리적 내용 → 실제로 어떤 것을 의미하는지
```java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 한다.
     * @serial
     */
    private final String lastName;
 
    /**
     * 이름. null이 아니어야 한다.
     * @serial
     */
    private final String firstName;
 
    /**
     * 중간이름. 중간이름이 없다면 null
     * @serial
     */
    private final String middleName;
 
    ... // 나머지 코드는 생략
}
```
- 성명은 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성
- 앞 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영

논리적, 물리적 표현 다르면 적합 x
```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
 
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    // ... 생략
}
```
- 논리적으로 이 클래스는 일련의 문자열을 표현 
- 물리적으로는 문자열들을 이중 연결 리스트로 연결

> 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 아래의 4가지 문제

### 1) 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
- 위 예제에서 private 클래스인 Entry가 공개 API가 되어버린다.
- 다음 릴리스에서 내부 표현 방식을 바꾸더라도 StringList 클래스는 여전히 연결 리스트로 표현된 입력도 처리가능해야 함
=> 즉, 연결 리스트의 관련 코드를 절대 제거할 수 없다.
### 2) 너무 많은 공간을 차지하면서, 속도가 느려질 수 있다.
- 앞 예의 직렬화 형태는 직렬화 형태에 포함될 가치가 없는 연결 리스트의 모든 엔트리와 연결 정보까지 기록 
### 3) 시간이 너무 많이 걸린다.
- 그래프를 직접 순회할 수 밖에 없으므로 시간이 걸린다.
### 4) 스택 오버플로를 일으킬 수 있다.
- 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 작업은 중간 정도 크기의 객체 그래프에서도 자칫 스택 오버플로를 일으킬 수 있다.

## 합리적인 직렬화 형태
- transient 가 붙으면 직렬화 제외 대상
```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;
    
    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) {...}
    
    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     * 
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s) throws IOException {
     	//기본 직렬화를 수행한다.
        s.defaultWriteObject();
        s.writeInt(size);
        
        // 커스텀 역직렬화를 수행한다.
        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }
    
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        //기본 역직렬화를 수행한다.
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // 커스텀 역직렬화 부분
        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for(int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }

```

## 주의점
- 객체의 논리적 상태와 무관한 필드라고 확신하면 transient 한정자를 생략 
- 동기화 메커니즘을 직렬화에도 적용
- 직렬 버전 UID를 명시적으로 부여
   - 이렇게 하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제 X
   - 직렬버전 UID는 클래스의 명세가 변경되면 자동 생성된 값이 바뀌기 때문에 이부분도 주의

### reference
https://devfunny.tistory.com/861?category=895441
https://github.com/Meet-Coder-Study/book-effective-java/blob/main/12%EC%9E%A5/87_%EC%BB%A4%EC%8A%A4%ED%85%80_%EC%A7%81%EB%A0%AC%ED%99%94_%ED%98%95%ED%83%9C%EB%A5%BC_%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC_%EB%B0%95%EC%86%8C%EC%A0%95.md
https://jaehun2841.github.io/2019/03/17/effective-java-item87/#%EC%8A%A4%ED%83%9D-%EC%98%A4%EB%B2%84%ED%94%8C%EB%A1%9C%EB%A5%BC-%EC%9D%BC%EC%9C%BC%ED%82%AC-%EC%88%98-%EC%9E%88%EB%8B%A4