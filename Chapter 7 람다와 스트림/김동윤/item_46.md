# 스트림에서는 부작용 없는 함수를 사용하라
> - 오직 입력만이 결과에 영향을 주는 함수
- 순수 함수는 다른 가변 상태를 참조하지 않고, 함수 스스로 다른 상태를 변경 X

## 스트림 제대로 활용
### forEach
### forEach 연산은 스트림 계산 결과를 보고할 때(주로 print 기능)만 사용하고 계산하는 데는 쓰지 말자 ! 
- forEach는 연산 결과를 보여주는 역할을 가진 종단 연산
- forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산할 때는 사용하지 말자

### (+) Stream의 foreach 와 for-loop 는 다르다
- 모든 요소를 순회하는 stream.forEach()

> Stream의 forEach는 요소를 돌면서 실행되는 stream 연산의 최종 작업이다. 보통 System.out.println 메소드를 넘겨서 결과를 출력할 때 사용

> - Stream 자체를 강제적으로 종료시키는 방법 x
- 무언가 강제적인 종료 조건에 대한 로직이 있는 for-loop를 stream.forEach()로 구현한다면, 기존 for-loop에 비해 비효율이 발생

```java
//for-loop로 짠 경우
for (int i = 0; i < 100; i++) {
    if (i > 50) {
      break;
      //50번 돌고 반복을 종료한다.
    }
    System.out.println(i);
}
```

```java
IntStream.range(1, 100).forEach(i -> {
    if (i > 50) {
      return;
      // 각 수행에 대해 다음 수행을 막을 뿐, 
      // 100번 모두 조건을 확인한 후에야 종료한다.
    }
    System.out.println(i);
});
```
-  stream.forEach()를 사용하게 되면 동작은 정상적으로 할지 몰라도 for문에 비해 비효율 발생

```java
IntStream.range(1, 100)
        .filter(i -> i <= 50)
        .forEach(System.out::println);
```
- Stream은 지연 연산을 하기 때문에 100번 모두 검사를 하긴 하지만 Stream.forEach()의 올바른 사용은 위처럼 forEach()를 최종 연산으로만 사용하는 것 !! 
- 이 stream.forEach() 내에 로직이 들어가지 않더라도, 중간연산인 filter, map, sort 등을 통해 충분히 로직을 수행 가능

(+) stream forEach 잘못 쓰인 경우
- forEach 내부에 로직이 하나라도 더 추가된다면 동시성 보장이 어려워지고 가독성이 떨어질 위험!! 
- 본래 forEach는 스트림의 종료를 위한 연산
- 로직을 수행하는 역할은 Stream을 반환하는 중간연산이 해야하는 일

```java
public void validateInput() {
    List<String> names = splitInputByComma();
        if (CollectionUtils.isEmpty(names)) {
            throw new IllegalArgumentException(LENGTH_ERROR_MESSAGE);
        }
    names.stream()
         .forEach(Input::validateNameLength);
}
```
```java
pieces.keySet()
      .forEach(
           positionKey -> model.addAttribute(
               positionKey,pieces.get(positionKey)));
```
### 수집기(collector)
- 스트림의 원소들을 축소해서 객체 하나에 모아주는 역할
- toList(), toSet(), toCollection(collectionFactory)
   - toList() : 스트림의 원소를 List에 모아줌
   - toMap() : 스트림 원소를 Key-Value 형태로 재생산, 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 사용하기 적합 `<toMap(keyMapper, valueMapper)>`
   - toMap(keyMapper, valueMapper)은 각 Key에 매핑되는 keyMapper(함수)와 Value에 매핑되는 valueMapper(함수)를 인수로 받음

```java
// toMap(keyMapper, valueMapper) 예제
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(
        toMap(Object::toStringg, e -> e));
```
- groupingBy
   - 분류 함수를 입력받고 원소들을 카테고리별로 모아 놓은 Map을 반환
- joining
   - 원소들을 연결하는 역할

## 출처
https://tecoble.techcourse.co.kr/post/2020-05-14-foreach-vs-forloop
https://blog.jooq.org/3-reasons-why-you-shouldnt-replace-your-for-loops-by-stream-foreach