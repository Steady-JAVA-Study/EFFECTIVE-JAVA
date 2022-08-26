#  일반적으로 통용되는 명명 규칙을 따르라
- 자바의 명명 규칙은 크게 철자와 문법 두 범주로 나뉘지

## 철자 규칙

> 패키지와 모듈	 : org.junit.jupiter.api, com.google.common.collect
클래스와 인터페이스 :	Stream, FutureTask, LinkedHashMap, HttpClient
메서드와 필드	: remove, groupingBy, getCrc
상수 필드 :	MIN_VALUE, NEGATIVE_INFINITY
지역 변수 :	i, denom, houseNum
타입 매개변수 :	T, E, K, V, R, U, V, T1, T2

### 패키지
- 패키지와 모듈 이름은 각 요소를 점(.)으로 구분하여 계층적으로 작성
- 요소들은 모두 소문자 알파벳 혹은 (드물게)숫자로 이뤄짐
- 조직 바깥에서도 사용될 패키지라면 조직의 인터넷 도메인 이름을 역순으로 사용
- 예외적으로 표준 라이브러리와 선택적 패키지들은 각각 java, javax(표준 확장 패키지)로 시작
`com.google, edu.cmu, org.eff`

### 클래스, 인터페이스
- 클래스와 인터페이스의 의름은 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작 `List, Map, ArrayList`
- 여러 단어의 첫 글자만 딴 약자나 max, min처럼 통용되는 줄임말을 제외하고는 단어를 줄여 쓰지 않도록 한다.
- 약자의 경우 첫글자만 대문자로 할지 전체를 대문자로 할지 살짝 논란이 있다. 첫 글자만 대문자로 쓰는 프로그래머가 훨씬 많다. - 전체가 대문자인경우 시작과 끝을 명확히 알 수 없다.
`AWT, Awt -> AWTUtils, AwtUtils
HTTPURL, HttpUrl`

### 메서드, 필드 
- 메서드와 필드 이름은 첫 글자를 소문자로 쓴다는 점만 빼면 클래스의 명명 규칙과 같다.
- 첫 단어가 약자라면 단어 전체가 소문자여야한다.
- 상수 필드는 예외, 단어를 모두 대문자로 쓰며 단어 사이는 밑줄로 구분

### 타입 매개변수

타입 매개변수 이름은 보통 한 문자로 표현
대부분 다음의 다섯 가지중 하나 ! 

- 컬렉션 원소의 타입 E
- 맵의 키와 값 K, V
- 예외 X(eXeption)
- 메서드 반환 타입 R
_____________

## 문법 규칙

### 클래스, 인터페이스

### 메서드 규칙
> - 어떤 동작을 수행하는 메서드의 이름은 동사나 (목적어를 포함한)동사구로 작성 `append, drawImage`
- boolean값을 반환하는 메서드라면 보통 Is나 드물게 has를 사용하여 명사, 명사구, 형용사와 결합해서 사용 `isDigit, isProbablePrime, isEmpty, isEnabled, hasSibings`
- 반환 타입이 boolean이 아니고 해당 인스턴스의 속성을 반환하는 메서드의 이름은 보통 명사, 명사구 혹윽 get으로 시작하는 동사구로 작성 `size, hashCode, getTime`




## 결론
- 표준 명명 규칙을 체화하여 자연스럽게 베어 나오도록 하자.

## REFERENCE 
https://jgrammer.tistory.com/