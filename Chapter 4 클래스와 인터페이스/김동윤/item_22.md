# 인터페이스는 타입 정의 용도로만 사용해
## 인터페이스 :  자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할
- 클래스가 어떤 인터페이스를 구현한다는 것 : 
자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 알려주는 것
=> 인터페이스는 오직 이 용도로만 사용해야 한다.

## 맞지 않는 상수 인터페이스 (상수 필드만 가득찬 인터페이스)

```java
public interface PhysicalConstants {

    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
    
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
    
}
```
- 사용하지 않을 수도 있는 상수를 포함하여 모두 가져오기 때문에 계속 가지고 있어야 함
- 다음 릴리즈에서 위 상수들을 안쓰더라도 바이너리 호환성을 위해 여전히 구현하고 있어야 함
- final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간이 그 인터페이스가 정의한 상수들로 오염되어 버림
- 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위
- 클래스가 어떠한 상수 인터페이스를 사용하든 이는 클라이언트에게 중요한 정보가 아니며, 오히려 혼란
-> 내부구현임에도 불구하고 클라이언트가 이 상수들에 종속되게 됨
-> 이 상수들이 더는 쓰이지 않더라도 바이너리 호환성을 위해 이 상수 인터페이스를 구현하고 있어야 한다
- 수 인터페이스를 구현하는 것은 내부 구현을 클래스의 API로 노출하는 행위

- 상수를 공개할 목적이라면 특정 클래스나 인터페이스 자체에 추가
- 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개
```java
public enum Day{ MON, TUE, WED, THU, FRI, SAT, SUN};
```

- 아니라면, 아래와 같이 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개
```java
public class PysicalConstants{
      private PysicalConstants(){}; // 인스턴스화 방지
      public static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      
      public static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```
## 결론
- 인터페이스는 타입을 정의하는 용도로만
- 상수 공개용 수단으로 사용 X

## 출처
https://dahye-jeong.gitbook.io/java/java/effective_java/2021-02-13-use-interface-to-define-type