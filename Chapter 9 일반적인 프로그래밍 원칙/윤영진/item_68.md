# ITEM 68. 일반적으로 통용되는 명명 규칙을 따르라

## 철자 규칙
### Package 명명 규칙

- 점(.)으로 구분하여 계층적으로 짓는다.
- 모두 소문자 알파벳 혹은 숫자로 구성한다.
- 조직 바깥에서도 사용될 패키지라면, 인터넷 도메인 이름을 역순으로 사용한다.
  - google.com -> com.google
  - 예외적으로 표준 라이브러리와 선택적 패키지들은 각각 java와 javax로 시작한다.
- 점(.)으로 구분되는 각 요소는 보통 8자 이하 짧은 단어/약어로 한다.
  - GlobalPositionSystem -> gps
  - utilities보다 util처럼 의미가 통하는 약어를 추천한다.
- 필요할 경우 계층을 나눠서 사용한다.
  - deliveryDriver -> delivery.driver

### Class & Interface 명명 규칙

- 하나 이상의 단어로 구성하며, 각 단어는 대문자로 시작한다. 
  - ex. List, FutherTask
- 여러 단어의 첫 글자만 딴 약어나 max, min처럼 널리 통용되는 줄임말을 제외하고 단어를 줄여쓰지 않는다.

### Method & Field 명명 규칙

- 첫 글자를 소문자로 작성한다.
- 단, 첫 단어가 약어라면 그 단어 전체가 소문자여야 한다.
- 상수 필드는 전체를 대문자로 한다. (static final)
  - 단어 사이에는 밑줄로 구분한다.
- 지역변수는 약어를 조금 더 적극적으로 사용해도 무방하다.
  - deliveryDriverForm -> form

### 타입 매개변수 명명 규칙 

- 보통 한 문자로 표현한다.
- 컬렉션 원소 타입 E, 맵의 키와 값은 K, V, 예외는 X, 메서드의 반환 타입은 R
- 그 외의 임의 타입 T, U, V 혹은 T1, T2, T3

## 문법 규칙

1. 객체를 생성할 수 있는 클래스
   - 단수 명사 or 명사구 
   - ex. Thread, PriorityQueue, ...
2. 객체를 생성할 수 없는 클래스
   - 복수
   - ex. PatternUtils, Collectors, Collections, ...
3. 인터페이스
   - 클래스와 동일 or ~able or ~ible
   - ex. Collection, Comparator, Runnable, ...
4. 애너테이션
   - 자유롭게
5. 동작을 수행하는 메서드
   - 동사 or 동사구
   - ex. append, drawImage, add, ...
6. boolean 값을 반환하는 메서드
   - is~ or has~
   - is가 보통이나 has도 있다.
   - ex. isDigit, isEmpty, hasText, ...
7. 반환하는 타입이 boolean이 아닌 메서드 
   - 명사 or 명사구 or get~
   - ex. size, hashCode, getTime
8. 객체의 타입을 바꿔서 다른 객체를 반환하는 메서드
   - to($Type)
   - ex. toString, toArray, ...
9. 객체의 내용을 다른 뷰로 보여주는 메서드
   - as($Type)
   - ex. asList, ...
10. 객체의 값을 기본 타입 값으로 변환하는 메서드
    - ($type)Value
    - ex. intValue, ...
11. 정적 팩터리 메서드
    - ex. from, of, valueOf, instance, getInstance, ... 