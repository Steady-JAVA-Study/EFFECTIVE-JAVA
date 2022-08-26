# 리플렉션보다는 인터페이스를 사용하라

## what is reflection ? 
- 난 리플렉션을 써본 적이 없음

> **리플렉션**
: 객체를 통해 클래스의 정보를 분석해 내는 프로그램 기법
- 구체적인 클래스 타입을 알지 못해서 그 클래스의 메소드와 타입 그리고 변수들을 접근할 수 있도록 해주는 자바 API 
- 구체적인 클래스 타입을 모를때 사용하는 방법
- 컴파일 시간이 아닌 실행 시간에 동적으로 특정 클래스의 정보를 추출할 수 있는 프로그래밍 기법

## 리플렉션은 언제 사용?

- 동적으로 클래스를 사용해야할 때 필요 
   > - 동적언어 : 런타임 시점에 타입을 결정 ex) Javascript, Python, Ruby ..
   - 정적언어 : 컴파일 시점에 타입을 결정 ex) Java, C, C++  
- 작성 시점에는 어떠한 클래스를 사용해야 할지 모르지만 런타임 시점에서 클래스를 가져와서 실행해야하는 경우 필요 
- 대표적으로는 Spring 프레임워크의 어노테이션 같은 기능들이 리플렉션을 이용하여 프로그램 실행 도중 동적으로 클래스의 정보를 가져와서 사용


## 리플렉션은 어떤 정보를 가져올 수 있음?
- Class
- Constructor
- Method
- Field
## 리플렉션 장점
- 런타임 시점에서 클래스의 인스턴스를 생성
- 접근 제어자와 관계 없이 필드와 메소드에 접근해, 필요한 작업을 수행할 수 있는 유연성 가진다.

## 리플렉션 단점
- 런타임 시점에서 인스턴스를 생성하므로 컴파일 시점에서 해당 타입을 체크할 수 없다.
- 런타임 시점에서 인스턴스를 생성하므로 구체적인 동작 흐름을 파악하기 어렵다.
- 리플렉션을 이용하면 코드 dirty
- 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 slow
- 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취하기 가능 
-  일반 메서드 호출보다 리플렉션을 이용한 메서드 호출이 훨씬 slow

## 리플렉션 사용 이유
- 리플렉션 API를 통해 런타임 중, 클래스 정보에 접근하여 클래스를 원하는 대로 조작 가능
   -  심지어 private 접근 제어자로 선언한 필드나 메소드까지 조작이 가능
- 프레임워크와 같이 큰 규모의 개발 단계에서는 수많은 객체와 의존 관계를 파악하기 어려움 
=> 이때 리플렉션을 사용하면 동적으로 클래스를 만들어서 의존 관계를 맺어줄 수 있다 ~ 오호

## 리플렉션 사용 예
- Spring의 Bean Factory를 보면, @Controller, @Service, @Repository 등의 어노테이션만 붙이면
=> Bean Factory에서 알아서 해당 어노테이션이 붙은 클래스를 생성하고 관리
- 개발자는 Bean Factory에 해당 클래스를 알려준 적이 없는데, 이것이 가능한 이유는 바로 리플렉션 덕분
- 런타임에 해당 어노테이션이 붙은 클래스를 탐색하고 발견한다면, 리플렉션을 통해 해당 클래스의 인스턴스를 생성하고 필요한 필드를 주입하여 Bean Factory에 저장하는 식으로 사용
- but 캡슐화를 저해하므로 꼭 필요한 상황에서만 사용하는 것이 good

& 

https://blog.gangnamunni.com/post/Annotation-Reflection-Entity-update/

_____________

> 따라서
생성은 리플렉션, 참조는 인터페이스
리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자. 
  
```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.util.Arrays;
import java.util.Set;

public class ReflectiveInstantiation {
  
@SuppressWarnings("unchecked")
public static void main(String[] args) {


    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
    	// 첫번째 인수로 클래스 확정 짓기 
      cl = (Class<? extends Set<String>>) 
    		  Class.forName(args[0]); 
      // "java.util.HashSet" 같은 것이 넘어올 것 
      // 넘어오는 것으로, 런타임 시점에 즉석 클래스 생성하는 것 ! 
    } catch (ClassNotFoundException e) {
      fatalError("클래스를 찾을 수 없습니다.");
    }

    // 위에서 확정 지은 클래스로부터 생성자를 얻기
    Constructor<? extends Set<String>> cons = null;
    try {
      cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
      fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }

    // 집합의 인스턴스를 만든다.
    Set<String> s = null;
    try {
      s = cons.newInstance();
    } catch (IllegalAccessException e) {
      fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
      fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
      fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
      fatalError("Set을 구현하지 않은 클래스입니다.");
    }

    // 생성한 집합을 사용한다.
    // 두번째 이후 인수들을 Set 저장 후 화면에 출력
    s.addAll(Arrays.asList("a","b","c").subList(1, 3));
    System.out.println(s);
  }

  private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
  }
}
  
```

> 1) Set<String> 인터페이스의 인스턴스를 생성한다. 정확한 클래스는 명령줄의 첫번째 인수로 확정한다.
2) 생성한 집합(Set)에 두번째 이후의 인수들을 추가한 다음 화면에 출력한다.
3) 첫번째 인수와 상관없이 이후의 인수들을 추가한 다음 화면에 출력한다.
4) 반면, 이 인수들이 출력되는 순서는 첫번째 인수로 지정한 클래스가 무엇이냐에 따라 달라진다.
  - HashSet 이라면 순서가 무작위 / TressSet을 지정하면 순서가 정렬되어 알파벳 순서로 출력될 것

> 위 코드의 한계
- (1) 리플렉션 없이 생성했다면 컴파일 타임에 잡아낼 수 있는 예외지만, 런타임 시 총 여섯가지나 되는 예외를 던지기 가능 
  -  인스턴스를 리플렉션 없이 생성했다면 컴파일 타임에 잡아낼 수 있었을 예외임
- (2) 클래스 이름만으로 인스턴스를 생성해내기 위해 25줄이나 되는 코드를 작성
  - 리플렉션이 아니라면 생성자 호출 한줄로 끝났을 것

________________________________
  
## 결론
### 리플렉션 사용 상황
- 드물긴 하지만, 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할때 적합
- 버전이 여러개 존재하는 외부 패키지를 다룰 때 유용
- 가동할 수 있는 최소한의 환경, 즉 주로 가장 오래된 버전만을 지원하도록 컴파일한 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근하는 방식을 사용 가능 
   - 이렇게 하려면 접근하려는 새로운 클래스나 메서드가 런타임에 존재하지 않을 수 있다는 사실을 반드시 감안해야한다. 

## REFERENCE
https://kdg-is.tistory.com (리플렉션)
https://dublin-java.tistory.com/53 (리플렉션)
https://devfunny.tistory.com/646