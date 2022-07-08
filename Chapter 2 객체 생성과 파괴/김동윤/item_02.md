
# 생성자에 매개변수가 많다면 빌더를 고려하라
- 매개변수 개수가 많아지면 클라이언트 코드 작성, 읽기 어려움 
- 엉뚱한 매개변수 건네주기 가능 

## 빌더 객체를 만드는 방법

>0 0) public static class Builder 로 해당 클래스 내에 선언
1) 빌더 객체를 사용하려면 가장 먼저 필수 파라미터를 입력하도록 한다.
2) 옵셔널 파라미터는 선택적으로 호출되도록 한다.
3) 옵셔널 세팅 작업이 완료되면 완성 메서드를 호출한다.
4) 객체는 반드시 해당 객체의 Builder 객체로만 생성할 수 있도록 한다.


### (1) builder 코드 예시
```java
public class Member {
    private final String id;    // required
    private final String name;  // required
    private final String email; // optional
    private final int height;   // optional
    private final int weight;   // optional


    public static class Builder {
        // 필수 파라미터
        private final String id;
        private final String name;

        // 옵셔널 파라미터
        private String email;
        private int height;
        private int weight;

        // 1. 빌더 객체를 사용하려면 가장 먼저 필수 파라미터를 입력하도록 한다.
        public Builder(String id, String name) {
            this.id = id;
            this.name = name;
        }

        // 2. 옵셔널 파라미터는 선택적으로 호출되도록 한다.
        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder height(int height) {
            this.height = height;
            return this;
        }

        public Builder weight(int weight) {
            this.weight = weight;
            return this;
        }

        // 3. 옵셔널 세팅 작업이 완료되면 완성 메서드를 호출한다.
        public Member build() {
            return new Member(this); 
        }
    }
  
    // 4. 객체는 반드시 해당 객체의 Builder 객체로만 생성할 수 있도록 한다.
    private Member(Builder builder) {
        id = builder.id;
        name = builder.name;
        email = builder.email;
        height = builder.height;
        weight = builder.weight;
    }
}
```
- 생성자는 private이며 setter 메서드는 존재 X 
```
Member member1 = new Member.Builder("ma-id", "ma-name")
                .height(100)
                .build();
```

## 빌더 패턴은 클래스 계층 구성에도 적합
```
public abstract class Pizza {
	public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;

	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}

		abstract Pizza build(); //피자 빌드 함수 

		protected abstract T self(); // 하위 클래스에서 이 메서드를 오버라이드 해서 this를 반환
	}

	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}
}
```
- 각 하위 클래스 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스 반환하도록 선언
- 하위 클래스 메서드가 상위 클래스 메서드가 정의한 반환 타입이 아닌 , `하위 타입 반환하는 기능`을 	`공변 반환 타이핑`이라고 한다,
=> 클라이언트는  형변환에 신경쓰지 않고 빌더 사용 가능~!
```java
public class NyPizza extends Pizza {

	public enum Size { SMALL, MEDIUM, LARGE }
	private final Size size;


// NyPizza를 return 해준다. 
	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;

		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}

		@Override
		public NyPizza build() {
			return new NyPizza(this);
		}

		@Override
		protected Builder self() {
			return this;		
		}

		public NyPizza(Builder builder) {
			super(builder);
			size = builder.size;
		}
	}
}
```