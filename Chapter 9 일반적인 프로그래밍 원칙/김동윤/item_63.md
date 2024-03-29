# 문자열 연결은 느리니 주의하라

## 문자열 연결 연산자 (+)
- 문자열 연결 연산자(+)는 여러 문자열을 편리하게 합쳐주지만, 본격적으로 사용하면 성능 저하를 감내하기 어려움

- 문자열 연결 연산자로 문자열 n개를 잇는 시간은 `n^2` 에 비례 (문자열은 불변 [아이템 17] 이라서 두 문자열 연결 시 양쪽 내용을 모두 복사해줘야해서)

```java
//문자열 연결을 잘못 사용한 예 - 느리다
public String statement() {
    String result = "";
    for(int i=0; i<items(); i++) {
        result += getItem(i);  //문자열 연결
    }
    return result;
}
```

품목이 많을 경우 이 메서드는 심각하게 느려진다.

## StringBuilder
StringBuilder를 사용하면 문자열 연결 성능이 크게 개선됨

```java
//연결 성능이 크게 개선됨
public String statement2() {
    String result = new StringBuilder(numItems() * LINE_WIDTH));
    for(int i=0; i<items(); i++) {
        result.append(getItem(i));  //문자열 연결
    }
    return result.toString();
}
```

## 결론
- 성능에 신경 써야하고 많은 문자열을 연결해야한다면 문자열 연결 연산자(+)를 피하자.

- 대신 StringBuilder의 append 메서드를 사용하라.

## 출처 
https://codingwell.tistory.com/145?category=1013497