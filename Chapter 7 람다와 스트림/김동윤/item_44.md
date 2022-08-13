> - 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다.
- 이를 대체하는 현대적 해법은 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것이다.
- 이 내용을 일반화하면 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다.

- 이자바 표준 라이브러리에는 같은 모양의 인터페이스가 준비되어 있다.
- 이java.util.function 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨있다.
- **함수형 인터페이스에 필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용**

## 함수형 인터페이스 
> 하나의 추상 메서드만 가지고 있는 인터페이스

```java
public interface MyFunctionalInterface {
    void execute();
}
```

# 불필요한 함수형 인터페이스 - 대신 표준 함수형 인터페이스를 사용해라! 

> - `java.util.function`  라는 패키지에서 다양한 용도의 표준 함수형 인터페이스를 제공
- 따라서 용도에 맞는 게 있다면, 직접 구현하기보다는 표준 함수형 인터페이스를 활용

# 표준 함수형 인터페이스
- java.util.function은 총 43개의 함수형 인터페이스를 제공한다. 그 중 다음 기본 인터페이스 6개만 기억하면 다른 인터페이스를 유추 easy 

## 1. `UnaryOperator<T>`
- T타입 인수 하나를 받아서 T 타입을 반환하는 함수형 인터페이스
```java
public static void main(String[] args) {
	UnaryOperator<Integer> plus10 = (i) -> i + 10;
	UnaryOperator<Integer> multiply2 = (i) -> i * 2;

	plus10.andThen(multiply2).apply(2); // (10 + 2) * 2 = 24
	plus10.compose(multiply2).apply(2); // (2 * 2) + 10 = 14
}
```

## 2. `BinaryOperator<T>`
- T 타입 인수 두 개를 받아서 T 타입을 반환하는 함수형 인터페이스
```java
public static void main(String[] args) {
	BinaryOperator<Integer> plus = (a,b) -> a + b;
	System.out.println(plus.apply(10, 20)); // 30 출력
}
```

## 3. `Predicate<T>` -> querydsl 검색 시 사용
- T타입을 받아서 boolean을 리턴하는 함수형 인터페이스
```java
public static void main(String[] args) {
	Predicate<String> isHello = (s) -> s.startWith("Hello");
	isHello.test("Hi"); // false
	isHello.test("Hello"); // true
}
```

(ex)
```java

    @Override
    public Page<CrImportanceResponse> findAllByCondition(CrImportanceReadCondition cond) {
        Pageable pageable = PageRequest.of(cond.getPage(), cond.getSize());
        Predicate predicate = createPredicate(cond);
        return new PageImpl<>(fetchAll(predicate, pageable), pageable, fetchCount(predicate));
    }


    private List<CrImportanceResponse> fetchAll(Predicate predicate, Pageable pageable) {  
        return getQuerydsl().applyPagination(
                pageable,
                jpaQueryFactory
                        .select(constructor(
                                CrImportanceResponse.class,
                                crImportance.id,
                                crImportance.name
                        ))
                        .from(crImportance)
                        .where(predicate)
                        .orderBy(crImportance.id.desc())
        ).fetch();
    }

    private Long fetchCount(Predicate predicate) { 
        return jpaQueryFactory.select(
                        crImportance.count()
                ).from(crImportance).
                where(predicate).fetchOne();
    }

    private Predicate createPredicate(CrImportanceReadCondition cond) { // 8
        return new BooleanBuilder();
    }

```


## 4. `Function<T, R>` : 리턴 타입 다름
- T 타입을 받아서 R 타입을 리턴하는 함수형 인터페이스
- 인수와 **리턴 타입 다름**
```java
public static void main(String[] args) {
	Function<Integer, String> print10 = (s) -> String.valueOf(s);
	System.out.println(print10 .apply(10)); // "10" 출력
}
```

## 5. `Supplier<T>` : 인수를 받지 않고 값 반환
- T 타입의 값을 제공하는 함수형 인터페이스로 공급자라고도 불린다. 인수를 받지 않고 값을 반환
```java
public static void main(String[] args) {
	Supplier<Integer> get10 = () -> 10; // 10을 리턴하겠다!
	get10.get();
}
```

## 6. `Consumer<T>`
- T타입을 받아서 아무 값도 리턴하지 않는 함수형 인터페이스다. 소비자
```java
public static void main(String[] args) {
	Consumer<Integer> printT = (i) -> System.out.println(i);
	printT.accept(10); // 10 출력
}
```

## (+) 7. `BiFunction<T, U, R>` 
- 두 개의 입력 값(T, U)를 받아서 R 타입을 리턴하는 함수형 인터페이스
```java
public static void main(String[] args) {
	BiFunction<Integer, Integer, Integer> plus = (a, b) -> a + b;
	System.out.println(plus.apply(10, 20)); // 30 출력
}
```

## 기본 함수형 인터페이스 사용 시 주의사항
- 기본 함수형 인터페이스에 박싱 된 기본 타입을 넣어 사용 X
- 동작에는 이상이 없지만 계산량이 많을 경우 성능 심각 저하 

## 구조적으로 같은 표준 함수형 인터페이스가 있더라도 직접 작성해야 할 때가 有
- 아래와 같은 경우에만 함수형 인터페이스 구현 여부 결정하자 ! 
1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
2. 반드시 따라야 하는 규약이 있다.
3. 유용한 디폴트 메서드를 제공할 수 있다.


## 결론 
- API를 설계할 때 람다도 염두에 두기
- 입력값과 반환값에 함수형 인터페이스 타입을 활용 
- 보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 NICE
- 흔치는 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있음 !

## 출처
https://hirlawldo.tistory.com/98 
https://jjingho.tistory.com/93
https://100100e.tistory.com/517