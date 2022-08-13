# 스트림은 주의해서 사용하라
- 다량의 데이터 처리를 돕고자 만들어짐

### 스트림 API가 제공하는 핵심 추상 개념 (STREAM, STREAM PIPELINE)
#### 1. 스트림
- 데이터 원소의 유한 또는 무한 시퀀스
#### 2. 스트림 파이프라인
- 원소들로 수행하는 연산 단계를 표현하는 개념
- 대표적인 스트림 소스로는 컬렉션, 배열, 파일, 정규표현식 패턴 매처, 난수 생성기 ㅇ

### 스트림 파이프라인 특징
- 소스 스트림으로 시작해 종단 연산으로 끝나며, 그 사이에 중간 연산이 들어갈 수 O
```java
public class Dummy {
    public static void main(String[] args) {
        List<String> nameList = Arrays.asList("a", "b", "c");

        nameList.stream() // 소스 스트림
                .filter(s -> s.startsWith("a")) // 중간 연산
                .forEach(System.out::println); // 종단 연산
    }
}
```

### 중간 연산
- 각 중간 연산은 스트림을 어떠한 방식으로 변환
- 각 원소에 함수를 적용하거나 특정 조건을 걸어 필터링할 수 있는 식
![](https://velog.velcdn.com/images/myway00/post/93cfa02d-05f6-45e2-a34c-641fd146bae3/image.png)

### 종단 연산
- 종단 연산은 중간 연산이 내놓은 스트림에 최후 연산
![](https://velog.velcdn.com/images/myway00/post/76f54349-33ac-4aee-8fa7-e3866a3981e3/image.png)

### 지연 평가(Lazy evaluation)
- 스트림 파이프라인은 지연 평가
- **평가는 종단 연산이 호출될 때 이뤄짐!!**
- 종단 연산에 쓰이지 않는 데이터 원소는 연산에 사용 X
- 종단 연산이 없으면 스트림 파이프라인은 아무 일 X

### 스트림 위험 
- 스트림을 제대로 사용하면 코드가 짧고 깔끔
- 잘못 사용하면 읽기 어렵고 유지보수가 어려워짐

```java
public class Anagrams {
    public static void main(String[] args) {
        List<String> dictionary = Arrays.asList("Dormitory", "Dirty Room", "Hot water", "Worth tea");

        dictionary.stream()
                .collect(groupingBy(word -> word.chars().sorted()
                                .collect(StringBuilder::new,
                                        (sb, value) -> sb.append((char)value),
                                        StringBuilder::append).toString()))
                .values().stream()
                .filter(group -> group.size() >= 2)
                .map(group -> group.size() +": " + group)
                .forEach(System.out::println);
    }
}
```

- 적절 스트림 사용 개선

```java
public class Anagrams {
    public static void main(String[] args) {
        List<String> dictionary = Arrays.asList("Dormitory", "Dirty Room", "Hot water", "Worth tea");

        dictionary.stream()
                .collect(groupingBy(Anagrams::alphabetize))
                .values().stream()
                .filter(group -> group.size() >= 2)
                .forEach(group -> System.out.println(group.size() +": " + group))
    }

    // 도우미 메서드
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);;
        return new String(a);
    }
}
```
- 모든 구현을 스트림으로 표현하는 것이 아닌 alphabetize() 메서드를 통해 스트림을 적절한 부분에만 사용
- 스트림 변수명을 이해하기 쉽도록 만듦
### char 값들을 처리할 때는 스트림을 사용을 삼가 ! 
- 자바에서는 char용 스트림을 제공 X
- char를 스트림 요소로 사용하면 char가 아닌 int값이 반환 

### 스트림을 제대로 사용하는 최선의 방법?
- 기존 코드는 스트림을 사용하도록 리팩터링 하되, 새 코드가 더 나아 보일 때만 반영


## 스트림 사용이 적절하지 않은 경우
스트림 파이프라인은 연산을 함수 객체로 표현한다. 함수 객체가 수행할 수 없는 계산 로직을 수행해야 하는 경우에는 스트림과 맞지 않는다고 할 수 있다.

### 1. 지역 변수를 읽거나 수정해야 하는 경우
람다에서는 final 혹은 effectively final인 변수만 읽을 수 있고, 지역 변수를 수정하는 게 불가능하다. 따라서 이러한 연산이 필요한 경우 코드 블럭을 사용해야 한다. 즉, 스트림 대신 반복 코드를 사용해야 한다.

### 2. return, continue, break를 수행하거나 메서드 예외를 던져야 하는 경우
람다에서는 위의 나열된 로직 중 어떠한 것도 수행할 수 없다. 따라서 이와 같은 로직이 필요하다면 스트림과 맞지 X

## 스트림 사용이 적절한 경우
1. 원소들의 시퀀스를 일관되게 변환 
2. 원소들의 시퀀스를 필터링 
3. 원소들의 시퀀스를 연산 후 결합 (더하기, 연결, 최솟값 구하기)
4. 원소들의 시퀀스를 컬렉션에 모음
5. 원소들의 시퀀스 중 특정 조건을 만족하는 원소를 찾기

## 스트림으로 처리하기 어려운 일
- 하나의 중간 연산을 거치면 원래의 값(원래의 스트림)을 잃는 구조

## 결론
- 스트림을 사용했을 때 깔끔하게 처리할 수 있는 경우가 있고, 반복 방식으로 더 직관적으로 표현할 수 있는 경우 有
- 많은 작업들이 이 둘을 적절히 조합해서 사용할 때 가장 NICE

- 어느 쪽을 선택하는 정해진 규칙 X
- 만약, 어느 방법이 더 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 선택해 사용



## 출처
https://jjingho.tistory.com/94