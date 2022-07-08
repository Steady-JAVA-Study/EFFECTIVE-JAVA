

# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 많은클래스가 하나 이상의 자원에 의존 경우 多
- 맞춤법 검사기를 구현한다 했을때, 아래와 같이 정적 유틸리티 클래스나 싱글톤으로 구현
1) 정적 유틸리티 클래스 방식

```java
public class SpellChecker {

    private static final Lexicon DICTIONARY = ...; //직접생성

    private SpellChecker() { //객체 생성 방지
    }

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
```
2) 싱글톤 방식

```java
public class SpellChecker {

    private final Lexicon dictionary = ...; //직접생성

    private SpellChecker(...) {...}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) {...}
    public List<String> suggestions(String typo) {...}
}
```
- 하지만, 위 두 방식은 유연하지 않고 테스트하기 어렵다. 
1) 왜냐하면 사용할 사전이 단 한가지라고 가정
2) 의존하는 객체를 직접 생성

## 의존 객체 주입 패턴을 사용하자~ 

```java

public class SpellChecker {

    private final Lexicon dictionary; //final로 불변을 보장하자

    public SpellChecker(Lexicon dictionary) { //생성자에 필요한 자원을 넘겨준다
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) {...}
    public List<String> suggestions(String typo) {...}
}
```

장점 1) 자원이 몇개든 의존관계가 어떻든 상관없이 잘 동작
장점 2) 유연하고 테스트가 쉽다

## 의존성 주입 패턴의 변형 : 생성자에 자원 팩터리를 넘겨주는 방식
- 팩터리 : 호출할 때마다 특정 타입의 인스턴스 반복해서 만들어주는 객체

- 자바 8의 `Supplier 인터페이스`가 리소스의 생성자에 전달하는 팩터리로 쓰기에 적합


  
```java
Mosaic create(Supplier<? extends Tile> tileFactory){..}
```
- Supplier<T> 를 입력으로 받는 메소드는 일반적으로 한정적 와일드카드 타입을 사용해 팩토리 타입 매개변수 제한
- 위 코드에서는 Tile의 하위클래스들이 Supplier에 올 수 있음을 알려주고 있는 와일드 클래스를 사용함 

```java
Mosaic flowerMosaic = mosaicCreator.create(() -> new Tile("꽃무늬"));
Mosaic checkMosaic = mosaicCreator.create(() -> new Tile("체크무늬"));
```
- 이를 사용해 클라이언트는 자신이 명시한 타입의 하위타입이라면 뭐든 생성 가능한 팩토리 넘기기 가능 

![](https://velog.velcdn.com/images/myway00/post/44549421-3158-47fc-8dfa-389b6adc668e/image.png)
- 팩토리도 자신이 생성해놓은 인스턴스 반환해주는 아이 => 따라서 이 클래스 내에서 또 직접 자원을 만들거나 직접 명시하지 않아도 된다는 점에서 위의 생성자에 필요한 자원 넘겨주는 것과 결이 비슷한 것 
  
## 스프링 의존 객체 주입~?
### 상황 
- 000Service에서는 원하는 객체를 위해 ClassificationRepository라는 자원이 필요함 
- 해당 자원을 내 필드에 데려오고 생성해야 함~! 

`000Service.java`

```java
@Service
@Transactional(readOnly = true)
public class NewItemService {

    private final NewItemRepository newItemRepository;
  // 스프링에서는 빈으로 등록된 NewItemRepository를 자동으로 외부에서 주입
////NewItemRepository newItemRepository = new NewItemRepository(); 일케 직접생성 안해두 됨! 
// 외부(IOC컨테이너) 에서 객체를 생성한 후 의존성을 주입해주기 덕분이지 

    public NewItemService( 
//이렇게 인스턴스 생성 시 생성자에 필요한 자원 넘겨주는 방식 
		NewItemRepository newItemRepository;
  ) { 
		this.newItemRepository = newItemRepository;
    }

public NewItemCreateResponse create(NewItemCreateRequest req) {
		// 생성자에서 이미 의존성 주입 됐으니깐 바로 갖다쓰면 완료~ 
        NewItem item = newItemRepository.save ~ ...


```
- lombok에서 제공하는 @RequiredArgsConstructor 로 필요한 자원을 생성자에서 만들어주면서~ 우리는 서비스 단에서 의존 객체 주입을 진행하고 있었던 것이다.
  

  
## > 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자