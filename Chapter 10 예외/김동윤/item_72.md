# 표준 예외를 사용하라

## 표준 예외 재사용의 장점
- 다른 사람이 API를 익히고 사용하기 쉬워 
- 낯선 예외를 사용하지 않으므로 코드의 가독성이 높아 
- 예외 클래스의 수가 적을수록 메모리 사용량도 줄고 클래스 로딩시간도 줄어 

## 많이 사용되는 예외
### IllegalArgumentException
- 메서드의 파라미터로 부적절한 값이 들어왔을 때 던지는 예외입니다.
- null이 들어오면 관례상 NullPointerException 예외를 사용합 
- 시퀀스 허용범위를 넘기는 경우 IndexOutOfBoundsException 예외를 사용 
### IllegalStateException
- 객체의 상태가 호출된 메서드의 수행에 부적합할 때 사용(초기화되지 않은 객체를 사용하려 할 때)
### ConcurrentModificationException
- 단일스레드에서 사용하려고 설게한 객체를 여러 스레드가 동시에 수정하려고할 때 사용 
- 동시 수정을 안정적으로 검출할 방법이 없기 때문에 일반적으로 문제가 생길 가능성 정도만 알려주는 역할로 쓰임
### UnsupportedOperationException
- 클라이언트가 호출한 메서드를 지원하지 않을 때 사용 
### ArithmeticException
- 복소수나 유리수를 다루는 개체를 다룰 때 사용 
- 10/0과 같은 연산을 수행하려는 경우 사용 
### NumberFormatException
- 복소수나 유리수를 다루는 개체를 다룰 때 사용 
### Integer 자리에 String이 오는경우 사용할 수 있습니ㅏㄷ.
- 예외 재사용시 주의사항
### Exception, RuntimeException, Throwable, Error는 직접 재사용하면 no no
- 이 클래스들은 추상 클래스라고 생각하길 바란다. 이 예외들은 다른 예외들의 상위 클래스이므로，즉 여러 성격의 예외들을 포괄하는 클래스이므로 안정적으로 테스트할 수 x

## reference
https://joeylee.tistory.com/53
https://sinau.tistory.com/m/133
