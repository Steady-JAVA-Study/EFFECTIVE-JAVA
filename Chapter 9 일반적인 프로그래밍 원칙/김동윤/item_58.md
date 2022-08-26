# 전통적인 for 문보다는 for-each 문을 사용하라

- for-each는 'enhanced-for statement' 로 '향상된 for 문' 을 의미
- 반복자나 인덱스 배열을 사용하지 않아 코드가 깔끔
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다.
- for-each 문은 컬렉션을 중첩해서 사용해야 할 때 그 이점이 더욱 BIGGER

## for-each 문을 사용할 수 없는 상황이 세 가지 존재한다.

### 1. 파괴적인 필터링
- 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 remove 메소드를 호출해야 한다.

### 2. 변형
- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

### 3. 병렬 반복
- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

```java
        olds.stream().forEach(i -> {

            i.setAttach_comment(
                    oldComment.size()==0?" ":
                    oldComment.get(
                            (olds.indexOf(i))
                    ).isBlank()?
                            " ":oldComment.get(
                            (olds.indexOf(i))
                    )
            );

```

### REFERENCE 
https://javabom.tistory.com 