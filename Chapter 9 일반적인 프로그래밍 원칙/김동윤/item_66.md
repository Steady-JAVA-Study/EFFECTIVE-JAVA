# 네이티브 메서드는 신중히 사용하라

## 네이티브 메서드 JAVA Native Method (JNI) ? 
![](https://velog.velcdn.com/images/myway00/post/a0fb0956-9d3f-4da4-9d89-70ba2c03efe5/image.png)
- 자바 프로그램에서 네이티브 메서드를 호출하는 기술을 JNI(Java Native Interface)

- 두 언어를 결합해서 사용해야 하는 경우가 존재하지
- JNI는 Java와 Java 이외의 언어로 만들어진 어플리케이션이나 라이브러리가 상호작용할 수 있도록 연결시켜주는 인터페이스
- 현재는 C/C++에 대한 호출만을 정확하게 지원현재는 C/C++에 대한 호출만을 정확하게 지원


## 네이티브 메서드 단점
- 네이티브 언어가 안전하지 않으므로 네이티브 메서드를 사용하는 애플리케이션도 메모리 훼손 오류로부터 안전 X
- 네이티브 언어는 자바보다 플랫폼을 많이 타 (자바는 BYTE CODE 만 있으면 자바 구동 환경에서 다 올릴 수 있는 플랫폼 독립 언어) 이식성 LOW
- 디버깅이 어렵다.
- 가비지 컬렉터가 네이티브 메모리는 자동으로 회수하지 못하고, 추적하지 못한다.
- 자바 코드와 네이티브 메서드를 넘나들때 비용이 추가 
- 네이티브 메서드와 자바 코드 사이에 접착코드(glue code) 를 작성해야 하는데, 귀찮고 가독성이 BAD 

## 결론
- 굳이 써야 한다면, 성능 개선 용도로만 써라. 
- 저수준(low-level) 자원이나 기존 라이브러리(legacy library)를 이용하기 위해 네이티브 메서드를 사용해야 한다면, 네이티브 코드는 가능하면 줄이고 광범위한 테스트를 거쳐라
- 네이티브 코드에 있는 아주 작은 버그라도 시스템 전체를 훼손 가능 

## reference
https://mommoo.tistory.com/71 (JNI)
https://songiam.tistory.com/74?category=796337