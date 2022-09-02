#  추상화 수준에 맞는 예외를 던지라
- 수행하려는 일과 관련 없어 보이는 예외가 튀어나오면 당황 
- 메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버릴 때 종종 일어나는 일

## 해결방법
### 1. 예외번역(Exception translation)
> 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다

```java
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) { // 저수준 예외 잡아
        throw new IndexOutOfBoundsException("인덱스: " + index);
        // 자신의 추상화 수준에 맞는 예외로 바꿔 던져
    }
}
```


### 2. 예외 연쇄(Exception chaning)
- 예외를 번역 할때 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는게 좋다
> 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식

```java
try {
    // 저수준 추상화를 사용
} catch (LowerLevelException cause) {
    // 저수준 예외를 고수준 예외에 실어 보낸다.
    throw new HigherLevelException(cause);
}

class HigherLevelException extends Exception { 
    HigherLevelException(Throwable cause) {
        super(cause); 
    }
}


출처: https://joeylee.tistory.com/54 [조이리:티스토리]
```


- 무턱대고 예외를 전파하기 보다는 가능하다면 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선 
- 차선책 : 아래 계층에서의 예외를 피할 수 없다면, 상위 계충에서 그 예외를 조용히 처리하여 문제를 API호출자에까지 전파하지 않는 방법이 있다. 로깅으로 남긴다

## 결론
 
- 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라. 
- 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 nice

## reference 
https://joeylee.tistory.com/54