# Serializable을 구현할지는 신중히 결정하라

## Serializable 구현

- 클래스 선언에 implements Serializable 붙이기

## Serializable 구현 단점, 문제
### Serializable을 구현하면 릴리스한 뒤에는 수정 어려움
- 직렬화된 바이트 스트림 인코딩(직렬화 형태)도 하나의 공개 API 
- 기본 직렬화 형태에서는 private와  package-private  인스턴스 필드들 마저  api로 공개하는 꼴-(캡슐화 깨짐)
- 직렬화가 클래스 개선을 방해 

### 버그와 보안 구멍이 생길 위험이 높다
- 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출 
### 해당 클래스의 신버전을 릴리스할때 테스트할 것이 늘어나
- 양방향 직렬화/역직렬화가 모두 성공하고 원래의 객체를 충실히 복제해내는지 반드시 확인 필요

### 직렬 버전 UID
- 스트림 고유 식별자로 모든 직렬화된 클래스는 고유 식별 번호를 부여
- serialVersionUID 라는 이름의 static final long필드로 이번호를 명시하지 않으면 시스템이 런타임에 암호 해시 함수를 적용해 자동으로 클래스 안에 생성해 넣어줌
- 그래서 위처럼 나중에 편의 메서드를 추가하는 것처럼 이들 중 하나라도 수정된다면 직렬 버전 UID 값도 변함
   - 쉽게 호환성이 깨져 런타임에 InvalidClassException이 발생
### 해당 클래스의 신버전을 릴리스할 때 테스트 할 것 늘어
- 직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화 할 수 있는지, 그 반대도 가능한지 검사 필요
- 따라서 테스트 할 양이 직렬화 가능 클래스의 수와 릴리스 횟수에 비례해 증가

## 주의사항
- static 멤버 클래스 : Serializable 구현 가능
- 익명 내부 클래스, 지역 내부 클래스 : Serializable 구현 불가능
## 결론
- 클 스의 여러 버전이 상호작용할 일이 없고 서버가 신뢰할 수 없는 데이터에 노출될 가능성 이 없는 등，보호된 환경에서만 쓰일 클래스가 아니라면 Serializable 구현은 아주 신 중히,,
- 상속할 수 있는 클래스라면 주의사항이 더욱 many

### reference
https://joeylee.tistory.com/38
https://songiam.tistory.com/85