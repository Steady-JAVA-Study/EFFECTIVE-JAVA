# 스레드 안전성 수준을 문서화하라

## 1. API 문서 synchronized 한정자
- 메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 x
- API 문서에 synchronized 한정자가 보인다고해서 이 메서드가 스레드 안전하다고 믿기 어려움
- 멀티 스레드 환경에서 안전하게 사용하려면 지원하는 스레드 안전성 수준을 정확히 명시!

## 2. 스레드 안전성 수준
### 불변(immutable)
- 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화가 필요없다.
   - String , Long, BigInteger
   - @Immutable
 
### 무조건적 스레드 안전(unconditionally thread-safe)
- 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다.
   - AtomicLong ,ConcurrentHashMap
   - @ThreadSafe
 
### 조건부 스레드 안전(conditionally thread-safe)
- 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다.
   - Collections.synchronized 래퍼 메서드가 반환한 컬렉션들
   - @ThreadSafe
 
### 스레드 안전하지 않음(not thread-safe)
- 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야한다.
   - ArrayList , HashMap
   - @NotThreadSafe
 
### 스레드 적대적(thread-hostil)
- 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다.
   - 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정 
   - 문제를 고쳐 재배포 하거나 사용자제 (deprecated) API로 지정 
 
## Conditionally Thread-safe의 문서화는 주의해서 써야해~ 
- 클래스의 스레드 안전성은 보통 클래스의 문서화 주석에 기재하지만 독특한 특성의 메서드라면 메서드의 주석에 기재하도록 한다.

- 반환타입만으로 명확히 알 수 없는 정적 팩터리라면 반환 객체의 스레드 안전성을 반드시 문서화 해야한다.

### REFERENCE
https://javabom.tistory.com/91