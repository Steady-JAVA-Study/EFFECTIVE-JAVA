# ITEM 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다.

자바독은 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.

javadoc 명령어 사용

```text
$ javadoc -d docs {file_name}
```

한글 사용시 UTF-8로 인코딩 필요

```text
$ javadoc -d docs *.java -encoding UTF-8 -charset UTF-8 -docencoding UTF-8
```

## 메서드용 문서화 주석

1. how가 아닌 what을 기술해야 한다. (어떻게 동작하는지가 아니라 무엇을 하는지 기술해야 한다.)
2. 클라이언트가 해당 메서드를 호출하기 위한 전제조건과 사후조건을 모두 나열해야 한다.
3. 전제조건은 @throws로 비검사 예외를 선언한다. 비검사 예외 하나당 전제조건 하나와 연결된다.
4. @param으로 그 전제조건에 영향받는 매개변수에 기술한다.

```java
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param  index index of element to return; must be
 *         non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index) {
    return null;
}
```

## javadoc 주석 태그

- `@param`
  - 전제조건에 영향받는 매개변수에 붙인다.
  - 모든 매개변수에 붙이는게 좋다.
  - 관례상 명사구를 쓰고, 마침표를 붙이지 않는다.

- `@return`
  - 반환타입이 void가 아니라면 붙인다.
  - void가 아니라도 메서드의 설명과 일치할 경우엔 생략해도 된다.
  - 관례상 명사구를 쓰고, 마침표를 붙이지 않는다.

- `@throws`
  - 비검사 예외를 선언한다.
  - 비검사 예외 하나당 전제조건 하나를 연결한다.
  - 관례상 마침표를 붙이지 않는다

- `{@code ...코드... }`
  - 태그로 감싼 내용을 코드용 폰트로 렌더링한다 -
  - 태그로 감싼 내용에 포함된 HTML요소나 다른 자바독 태그를 무시한다
  - <pre>{@code ...코드... }</pre> 와 같이 사용하면 여러줄로 된 코드도 작성 가능하다. 단 @을 쓸땐 탈출문자 붙여야 함
    
- `{@literal ... }`
  - <, >, &등의 HTML메타문자를 포함시킨다.
  - {@code ...코드... } 와 비슷하지만 코드 폰트로 렌더링하진 않는다

- 첫 문장은 주로 요약 설명이다
  - 한 클래스 혹은 한 인터페이스 안에 요약설명이 중복되면 안된다. (특히 오버로딩된 메서드들에서 특히 조심하자)
  - 마침표에 주의해야 한다.
