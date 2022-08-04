# ITEM 39. 명명 패턴보다 애너테이션을 사용하라

## 명명 패턴


> 명명 패턴?
>
> 메서드에 이름을 특정 이름으로 시작하게 만들어서, 해당 이름이 들어가면 실행하도록 만들기 위한 패턴

![image](https://user-images.githubusercontent.com/83503188/182889948-6e1fe57d-e8f1-4ed7-bb92-c4c4f481797c.png)


JUnit3 의 경우 테스트 케이스의 이름은 test 로 시작해야 한다.

test 로 시작해야 테스트 케이스로 인식한다.

TestCase 클래스를 상속받아야 한다.

```java
import junit.framework.*;

public class TestSample extends TestCase {
    public TestSample(String name) {
        super(name);
    }

    public void setUp() {
    }

    public void tearDown() {
    }

    // 명명패턴 - 테스트 케이스는 test 로 시작
    public void test01() {
        //테스트 코드
    }

    public static Test suite() {
        return new TestSuite(TestSample.class);
    }

    public static void main(String args[]) {
        TesrFinder.run(TestSample.class, args);
    }
}
[레필리아의 잡동사니::단위테스트 JUNIT(3)]
        (https://repilria.tistory.com/315)
```

위와 같은 명명 패턴은 다음과 같은 단점을 가진다.

1. 오타가 나면 인식을 못한다.
2. 의도한 곳에서만 쓰일 것이라 보장할 수 없다.
3. 명명 패턴에는 패턴 실행에 필요한 인자를 매개변수로 넘길 방법이 마땅치 않다.

## 애너테이션

![image](https://user-images.githubusercontent.com/83503188/182890304-32eac760-6dce-4aed-aa95-1021609203e1.png)

```java

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}


@Constraint(validatedBy = {EnumValidator.class})
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface Enum {

    String message() default "Invalid value. This is not permitted.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    Class<? extends java.lang.Enum<?>> enumClass();
    boolean ignoreCase() default false;

}
```

- @Retention : 애너테이션의 스코프를 결정한다.
- @Target : 애너테이션이 적용될 대상을 결정한다.

### 애너테이션을 사용한 프로그램 작성


```java
public class Sample {
    @Test
    public static void m1() {
    }

    public static void m2() {
    }

    @Test
    public static void m3() {
        throw new RuntimeException("fail");
    }
}
```

- m1 메서드는 @Test가 선언되었으므로 테스트에 통과할 것이다.
- m2 메서드는 @Test가 선언되지 않았기 때문에 테스트를 수행하지 않는다.
- m3 메서드는 @Test가 선언되었지만, 약속되지 않은 예외를 던지기 때문에 테스트에 실패할 것이다.

### 애너테이션을 처리하는 프로그램 작성

중요한 것은, 애너테이션을 사용하는 것 자체로는 해당 클래스에 직접적인 영향을 미치지 않는다. 마커 애너테이션은 표시일 뿐이다. 이 애너테이션은 이를 처리하는 프로그램에 추가적인 정보를 제공하기 위해 사용된다.

```java
public class EnumValidator implements ConstraintValidator<Enum, String> {

    private Enum annotation;

    @Override
    public void initialize(Enum constraintAnnotation) {
        this.annotation = constraintAnnotation;
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        boolean result = false;


        if (value == null && this.annotation.ignoreCase()) {
            result = true;
        } else if (value == null) {
            result = false;
        } else {
            Object[] enumValues = this.annotation.enumClass().getEnumConstants();
            if (enumValues != null) {
                for (Object enumValue : enumValues) {

                    if (value.equals(enumValue.toString())) {
                        result = true;
                        break;
                    }
                }
            }
        }

        return result;
    }

}
```

