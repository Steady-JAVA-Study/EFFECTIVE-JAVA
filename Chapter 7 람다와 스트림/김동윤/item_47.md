- 일련의 원소를 반환하는 메서드는 자바 7까지 반환 타입으로 Collection , Set , List 같은 컬렉션 인터페이스나, Iterable 이나 배열 을 사용 했다.

- BUT 자바 8에서 스트림이라는 개념이 들어오게 되었는데, 스트림은 반복(iteration)을 지원하지 않는다.

- SO ! 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.

- 만약 API를 스트림만 반환하도록 짜놓으면 반한된 스트림을 for-each로 반복하길 원하는 사용자는 불만 생기기 가능 ! 

-  Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, Iterable 인터페이스가 정의한 방식대로 동작
   - NEVER THE LESS : for-each로 스트림을 반복할 수 없는 이유는 Iterable을 확장 X 라서 
   
# 반환 타입으로는 스트림보다 컬렉션이 낫다
> 원소 시퀀스, 즉 일련의 원소를 반환하는 메서드는 수없이 많다.
- Collection
- Set
- List
- Iterable
- Array

## Stream에서 iterator 사용하기 (스트림을 반복하기 위한 방법)

### 어댑터 메서드
- 형변환 과정 NEEDED
- `Stream<E>`를 `Iterable<E>`로 중개해주는 어댑터를 생성해서 사용
- 어댑터 메서드를 사용하면 타입추론이 문맥을 잘 파악하여 형변환을 따로하지 안해도 됨

```java
// Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream){
    return stream::iterable;
}

// 어댑터 메서드를 사용해 반복
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // 처리 로직
}
```
### iterator에서 Stream 사용하기
- API가 Iterable만 반환하면 스트림을 사용하려는 사용자는 불만이 생김
- 댑터 메서드를 구현해 손쉽게 구현 가능
```java
public static <E> Stream<E> streamOf(Iterable<E> iterator) {
    return StreamSupport.stream(iterator.spliterator(),false);
}
```

## 그러므로 Collection을 반환 ! 
- Collection 인터페이스는 Iterable 하위타입 
- Collection 인터페이스는 stream 메서드도 제공 
- Therefore, Collection을 반환하는 것이 가장 NICE

## 결론

- 원소 시퀀스를 반환하는 메서드를 작성할 때, 스트림으로 처리하길 원하는 사용자와 반복으로 처리하길 원하는 사용자 둘다 만족 시키려고 노력해야 한다고 함~! 
- 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환 
   - OR 전용 컬렉션을 구현할지 고민 
- 컬렉션을 반환하는게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환~ 

### 출처 
https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/06/20/


______________________

## Iterator

> ITERATOR
- 어떤 컬렉션이라도 동일한 방식으로 접근이 가능하여 그 안에 있는 항목들에 접근할 수 있는 방법을 제공 (다형성) 
```java

public interface Iterator {

boolean hasNext(); 
// 읽어 올 요소가 남아있는지 확인하는 메소드
// 요소가 있으면 true,
// 없으면 false
Object next();
// 읽어 올 요소가 남아있는지 확인하는 메소드

void remove();
// next()로 읽어 온 요소를 삭제
}

```

> iterator
```java
Iterator<String> iter = list.iterator();

while(iter.hasNext()){ 
	System.out.println(iter.next());

}
```


### 1) 사용가능 객체
```java
ArrayList arr = new ArrayList();

  Vector<> vec = new Vector<>();

 LinkedList<Member>  link = new LinkedList<Member>();

 Set<String> st     = new Set<>(String);

 ....

 과 같은 Collection 객체들
 ```
 
 ### 2) Iterator객체 얻기
 
 ```java
 
 Iterator it0 = arr.iterator();  
 //vector의 요소를 순차 검색할 객체

  Iterator<> it1 = vec.iterator<>();

  Iterator<Member> it2 = link.iterator<Member>();

  Iterator<String> it3 = st.iterator<String>();

 ```
 
### 3) while 문으로 하나씩 객체 얻기

- hasNext() : 해당 iterator객체에 값 존재 여부(boolean)
- next() : iterator객체의 값 얻기.

```java
 while (it0.hasNext()){

    String st = (String)it0.next();

 }
```
