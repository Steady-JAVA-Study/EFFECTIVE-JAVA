
# 불필요한 객체 생성을 피하라

- 같은 기능을 하는 객체를 여러 번 사용하는 것 보다 객체 하나를 재사용 하는 것이 더 나을 때가 있음
- 불변 객체는 언제든 재사용 가능
- 비싼 객체들 중 재사용 가능한 것은 미리 생성해두는 것이 아주 좋다고 할 수 있다.
- 비싼 객체 == memory, disk I/O 에서 많은 용량을 잡아먹는 것 & Creating big array 등이 존재 
- 생성자 대신 정적 팩터리 메서드 제공 불변 클래스에서는 정적 팩터리 메서드 사용해 불필요 객체 생성 방지 가능 
- 생성자는 호출할 때마다 새로운 객체 만듦, but 팩터리 메서드는 nono 
```java
// slow
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}

// 미리 컴파일 해놓은 객체를 불러오기
private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}
```
- 성능 개선 위해서 불변인 인스턴스라면 클래스 초기화 과정에서 직접 생성해 캐싱해두고, 나중에 isRomanNumeral 메서드 호출될 때마다 이 인스턴스 재사용

____

## autoboxing 도 불필요 객체 만들어내는 것 
- 박싱된 기본(reference) 타입보다는 기본(primitive) 타입을 사용하기
- 오토박싱이 숨어들지 않게 주의하기 
- 오토박싱 : primitive -> reference 되는 과정 
```java
// 박싱
int i = 10;
Integer num = new Integer(i);
```
- 은밀한 오토박싱 때문에 아주 비효율적이고 느려진 코드의 example
```java 
public class AutoBoxingExample {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        Long sum = 0l;
        for (long i = 0 ; i <= Integer.MAX_VALUE ; i++) {
            sum += i; //Long sum ++ new Long(i); 로 오토박싱되고 있음
        }
        System.out.println(sum);
        System.out.println(System.currentTimeMillis() - start);
    }
}
```
=> 불필요한 오토박싱을 피하려면 박스 타입 보다는 프리미티브 타입을 사용

## 프로그래머의 insight
- [Item50]방어적 복사(defensive copy) 와 대비되는 내용이기 때문에 객체 생성을 해야하는 경우와 하지 않고 기존 것을 재사용해야 하는 부분은 알아서 파악 needed (혹은 성능 테스트를 직접 해서 비교해보아라)
- 새로 만든 생성자가 필요한 경우와 재사용 가능한 불변 객체를 사용하는 구분하는 능력을 키웁시다!

## 출처
이펙티브 자바 도서
https://github.com/Meet-Coder-Study/book-effective-java