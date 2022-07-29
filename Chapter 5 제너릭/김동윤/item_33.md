

# 타입 안전 이종 컨테이너를 고려하라

## container ?
- 기본 타입들은 List, Set, Queue, Map


__________________________________________________

- 제네릭은 단일원소 컨테이너에 흔히 쓰임(매개변수화할 수 있는 타입의 수가 제한됨)

- Map<Integer, String> 에는 String 타입의 값만 담기 가능 
- BUT 다른 타입들도 함께 담고 싶을 때는 ?
=> **타입 안전 이종 컨테이너** use ! (서로 다른 타입을 하나의 컨테이너에 안전하게 보관)

## 타입 안전 이종 컨테이너
- 각 타입의 Class 객체를 매개변수화한 키 역할로 사용~! (Class가 제네릭이므로 가능)
- 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공
=> 이런 설계 방식 > 타입 안전 이종 컨테이너 패턴 !! 

### 타입 안전 이종 컨테이너 -  Favorite class
```java
public class Favorite {

	private Map<Class<?>, Object> favorites = new HashMap<>();
	// 각 타입의 Class 객체를 매개변수화한 키 역할
    
	public <T> void putFavorite(Class<T> type, T instance) {
    // 각 타입의 Class 객체를 매개변수화한 키 역할
    
		favorites.put(Objects.requireNonNull(type), type.cast(instance));
	}
    
	public <T> T getFavorite(Class<T> type) {
    // 각 타입의 Class 객체를 매개변수화한 키 역할
    
		return type.cast(favorites.get(type));
	}
}
```
### CLIENT

```java
public static void main(String[] args) {
	Favorites f = new Favorites();
	
    // 저장하거나, 얻어올 때 Class 객체를 알려주면 된다 !! 
	f.putFavorite(String.class, "Java"); // Class<String> 
	f.putFavorite(Integer.class, 0xcafebabe); //Class<Integer> 
	f.putFavorite(Class.class, Favorites.class);

	String favoriteString = f.getFavorite(String.class);
	int favoriteInteger = f.getFavorite(Integer.class);
	Class<?> favoriteClass = f.getFavorite(Class.class);

	System.out.printf("%s %x %s\\n", favoriteString, favoriteInteger, favoriteClass.getName());
}

```

`result`

=> favorite 는 String, Integer, Class 인스턴스를 저장, 검색, 출력

```
Java
-889275714
Java cafebabe com.github.sejoung.codetest.generics.Favorites
```

### favorite class FEATURE
- EACH 타입의 Class 객체를 매개변수화한 키 역할
- Favorites 인스턴스는 타입 SAFE => String을 요청했는데 Integer를 반환하는 일은 절대 X
- 또한 모든 키의 타입이 제각각 => 일반적인 맵과 달리 여러 가지 타입의 원소를 SAVE OK! 

=> 따라서 Favorites는 타입 안전 이종 컨테이너

- 이 방식이 동작하는 이유 :
   - class의 클래스가 제네릭!
   - class 리터럴 타입은 Class 가 아닌 Class<T> 이다.
  
```
String.class의 타입 <=> Class<String> 
Integer.class의 타입 <=> Class<Integer> 
```
  
> 
  ### 타입 토큰 ! 
  - 컴파일타임 타입 정보와 런타임 타입 정보를 위해 메소드에서 주고 받는 class 리터럴 IS REFERED AS 타입 토큰 ! 
- **타입 토큰**(Type Token) =>  쉽게 말해 타입을 나타내는 토큰
- 클래스 리터럴이 타입 토큰으로서 사용된다.
(ex. Integer.class, String.class 는 class 리터럴이며 타입토큰은 `Class<Integer>`, `Class<String>`)
 
____________________________________

`PUT FAVORITE` & `GET FAVORITE`
  
```java
public class Favorites{
    private Map<Class<?>, Object> favorites = new HashMap<>();
    public  <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), instance);
    }
    public  <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
    }
}
```
  
### `putFavorite`   
- Favorites가 사용하는 private 맵 변수인 favorites 타입 
- Map<Class<?>, Object>
  - 1) KEY : 키가 와일드카드 타입
    - <=> 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻 
    -  다양한 타입을 지원하는 힘이 여기서 SPREAD
  - 2) VALUE : 단순히 Object
    - 키와 값 사이의 타입 관계를 보증 X
    - 하지만 우리는 이 관계가 성립함을 KNOW OO

### `getFavorite`
- 주어진 Class 객체에 해당하는 값을 favorites 맵에서 꺼냄
- 이 객체의 반환 타입은 Object 이나 T로 바꿔 반환해야함 
- Class 의 **cast 메서드** 통해 객체 잠조를 Class 객체가 가리키는 타입으로 동적 형변환
  
### cast : 형변환 연산자의 동적 버전
> 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지 검사 후 
맞으면 인스턴스 반환, 아니라면 ClassCastException 
  
```java
  
public T cast(Object obj) {
    if (obj != null && !isInstance(obj))
        throw new ClassCastException(cannotCastMsg(obj));
    return (T) obj;  
```
  
- 해당 예시 코드에서 favorites 맵 안의 값이 해당 키의 타입과 항상 일치함
- 따라서 예시코드에서 ClassCastException 발생 안함
- cast의 반환 타입 = Class 객체의 타입 매개변수 

<br>
  
## FAVORITE 제약 
  
### 1)  악의적인 클라이언트가 Class 객체를 제네릭이 아닌 _raw 타입_으로 넘기면 Favorites 인스턴스의 타입 안정성이 깨진다.

```java
f.putFavorites ((Class)Integer.class, "Integer의 인스턴스가 아닙니다.");
int favoritesInteger = f.getFavorite(Integer.Class);
```
- putFavorites를 호출 => 아무 문제 x
- getFavorite를 호출 => ClassCastException

=> Favorites가 타입 불변식을 어기는 일이 없도록 보장하려면 동적 형변환 use ! 

```java
public <T> void putFavorite(Class<T> type, T instance){
    favorites.put(Object.requireNonNull(type), type.cast(instance));
}
```
- putFavorite 메서드에서 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 확인 

  
  
### 2) 실체화 불가 타입에는 Favorites 클래스를 사용 X
-  String 이나 String[]은 사용할 수 있어도 
- List<String>이나 List<Integer>는 저장할 수 없다는 이야기  
  
<br>

- List<String>를 저장하려고 해도 컴파일 X
- List<String>용 Class 객체를 얻을 수 없기 때문
=>  (List<String>.class -> 문법 오류가 난다)
- 이유 : List<String> 이나 List<Integer> 나 List.class라는 같은 Class 객체를 공유 ;; 

## and ~~ 한정적 타입 토큰 : 타입 토큰  제한하고 싶을 때
- Favorites가 사용하는 타입 토큰은 비한정 적이므로, 어떤 Class 객체든 받아들인다. 
- sometimes~ 이 메서드들이 허용하는 타입을 제한 want =>  이때 한정적 타입 토큰 활용

(ex)  애너테이션 API =>한정적 타입 토큰을 사용
- 대상 요소에 달려 있는 애너테이션을 런타임에 읽어 오는 기능
```java
public <T extends Annotation>
    T getAnnotation(Class<T> annotationType);
```
- 이 메서드는 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환
- 없다면 null을 반환
#### 즉, 애너테이션된 요소는 그 키가 애너테이션 타입인, 타입 안전 이종 컨테이너 인 것 ! 
  
## 결론
- 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정
  
- BUT 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들기 가능!! 
  
- 타입 안전 이종 컨테이너는 **Class를 키**로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 함
  
  
## REFERENCE
- https://codingwell.tistory.com/115
- https://songiam.tistory.com/65