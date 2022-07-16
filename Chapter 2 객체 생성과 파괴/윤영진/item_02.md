# ITEM 2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩터리와 생성자는 선택적 매개변수가 많을 때 대응하기 어렵다. 이런 케이스에 `점층적 생성자 패턴`, `자바 빈즈 패턴`, `빌더 패턴`을 고려해볼 수 있다.

## 점층적 생성자 패턴(telescoping constructor pattern)

필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자, ... 형태로 선택 매개변수를 전부다 받는 생성자까지 늘려가는 방식이다.

```java
public class NutritionFacts {
	private final int servingSize;  // 필수
	private final int servings;     // 필수
	private final int calories;     // 선택
	private final int fat;          // 선택
	private final int sodium;       // 선택
	private final int carbohydrate; // 선택

	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, 0);
	}

		public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```

매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

## 자바 빈즈 패턴(JavaBeans pattern)

매개변수가 없는 생성자로 객체를 만든 후, 세터(setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식

```java
public class NutritionFacts {
	private int servingSize = -1;  // 필수
	private int servings = -1;     // 필수
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFacts() {}

	public void setServingSize(int val) { servingSize = val; }
	public void setServings(int val) { servings = val; }
	public void setCalories(int val) { calories = val; }
	public void setFat(int val) { fat = val; }
	public void setSodium(int val) { sodium = val; }
	public void setCarbohydrate(int val) {carbohydrate = val; }
}
```

```java
// 점층적 생성자 패턴에 비해 확장하기 쉽고, 인스턴스를 만들기 쉽고, 가독성이 좋아진다.
NutritinFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

인스턴스를 만들기 쉽고 가독성이 좋아진다. 

하지만, 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓이게 된다.

또한, 클래스를 불변(아이템 17)으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야만 한다.


## 빌더 패턴(Builder pattern)

점측정 생성자 패턴의 안전성과 자바 빈즈 패턴의 가독성을 겸비한 패턴

클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 그런 다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다. 마지막으로 매개변수가 없는 build 메서드를 호출해 타겟 객체를 얻는다. 

```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
		private final int servingSize;  // 필수
		private final int servings;     // 필수
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this,servingSize = servingSize;
			this.servings = servings;
		}

        public Builder calories(int val) {
            calories = val;
            return this;
        }

		public Builder fat(int val) {
			fat = val;
			return this;
		}

		public Builder sodium(int val) {
			sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	private NutirionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.fat;
		carbohydrate = builder.carbohydrate;
	}
}

```

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
	.calories(100)
	.sodium(35)
	.carbohydrate(27)
	.build();
```

### 빌더 패턴 - 계층적 빌더

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다

추상 클래스에는 추상 빌더를, 구체 클래스에는 구체 빌더를 갖게 한다.

```java
public abstract class Pizza {
	public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;

	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}

		abstarct Pizza build();

		protected abstract T self(); // 하위 클래스에서 이 메서드를 오버라이드 해서 this를 반환
	}

	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}
}
```

```java
public class NyPizza extends Pizza {
	public enum Size { SMALL, MEDIUM, LARGE }
	private final Size size;

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

```java
public class Calzone extends Pizza {
	private final boolean sauceInside;

	public static class Builder extends Pizza.Builder<Builder> {
		private boolean sauceInsice = false;

		public Builder sauceInside() {
			sauceInsdie = true;
			return this;
		}
		
		@Override
		public Calzone build() {
			return new Calzone(this);
		}

		@Override
		protected Builder self() {
			return this;
		}

		private Calzone(Builder builder) {
			super(builder);
			sauceInside = builder.sauceInside;
		}
	}
}
```

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
	.addTopping(SAUSAGE)
	.addTopping(ONION)
	.build();

Calzone pizza = new Calzone.Builder()
	.addTopping(HAM)
	.sauceInside()
	.build();
```

