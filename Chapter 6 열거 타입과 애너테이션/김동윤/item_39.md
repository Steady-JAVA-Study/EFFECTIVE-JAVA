# 명명 패턴보다 애너테이션을 사용하라

## 명명 패턴 - JUnit3
- JUnit3 의 경우 테스트 케이스의 이름은 test 로 시작 
- test 로 시작해야 테스트 케이스로 인식 
- TestCase 클래스를 상속받아야~

1. 오타가 나면 안된다.
- 실수로 이름을 tset~라고 지으면 그 테스트 메서드는 무시하고 지나가기 때문에 테스트 메서드가 제대로 실행됐는지 어쨌는지 모른다.

2. 올바른 프로그램 요소에서만 사용되리라 보증 할 방법 X
- 예컨데 TestSafetyMechanisms으로 JUnit에 던져줬다고 해보자. 
- 개발자는 이 클래스에 정의된 테스트 메서드들을 수행해 주길 기대하겠지만, JUnit은 클래스 이름에는 관심이 없다. 
- 이번에도 경고조차 출력하지 않고 개발자가 의도한 대로 테스트는 진행되지 않는다.

3. 프로그램 요소를 매개변수로 전달할 마땅한 방법 X
- 특정 예외를 던져야만 성공하는 테스트가 있을 때, 기대하는 예외의 타입을 매개변수로 전달해야 하는 상황이다.
- 예외의 이름을 테스트 메서드 이름에 덧붙이는 방법도 있지만, 보기에도 나쁘고 깨지기도 쉽다.

## JUnit4 부터는 애너테이션을 도입
### 마커 애너테이션 타입선언

```java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적메서드 전용이다.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{
}
```

```java
public class Sample {
    @Test
    public static void m1() {
        //성공
    }
  
    public static void m2() {
        //실행되지 않는다.
    }
    @Test
    public static void m3() {
        //실패
        throw new RuntimeException("실패");
    }
    public static void m4() {
        //실행되지 않는다.
    }
    @Test
    public void m5() {
        //잘못 사용한 예
        //static method가 아니다.
    }
    public static void m6() {
        //실행되지 않는다.
    }
    @Test
    public static void m7() {
        //실패
        throw new RuntimeException("실패");
    } 
    public static void m8() {
        //실행되지 않는다.
    }
}
```
> ### 마커 애너테이션
- @Test와 같은 애너테이션을 아무 매개변수 없이 단순히 대상에 마킹(Marking)한다는 뜻에서 마커 애너테이션 (Marker Annotation)
- @Test 애너테이션에 오타를 내면 컴파일 오류
- 클래스의 의미에 직접적으로 영향 X
- 그저 애너테이션에 관심있는 프로그램에게 추가 정보를 제고옹 

## 결론
- 애너테이션이 명명패턴을 이용할 때 보다 확실히 낫다.
- 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
- 명명패턴 단점 1 : 오타나면 인식못함
에너테이션은 오타나면 컴파일조차 안된다.
- 명명패턴 단점 2 : 이상한 데에 쓸 수도 있음
@Target, 또는 에너테이션 프로세서에서의 검사를 통해 제한가능
- 명명패턴 단점 3 : 실행에 필요한 요소 전달이 불가
에너테이션에 매개변수로 전달 가능
## REFERENCE 
https://jaehun2841.github.io/2019/02/04/effective-java-item39/#%EB%B0%98%EB%B3%B5-%EA%B0%80%EB%8A%A5-%EC%95%A0%EB%84%88%ED%85%8C%EC%9D%B4%EC%85%98-processor