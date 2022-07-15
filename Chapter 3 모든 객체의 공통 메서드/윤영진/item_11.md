# ITEM 11. equals를 재정의하려거든 hashCode도 재정의하라

**equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.** 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

> hashCode
> 
> 객체의 주소값을 변환하여 생성한 객체의 고유한 정수값이다.
> 
> Java의 모든 객체는 hashCode 메서드를 가진다. Java에서는 HashMap, HashSet 등 hash를 활용한 Collection들을 자주 볼 수 있는데 모두 hashCode를 기반으로 하고 있다.

![image](https://user-images.githubusercontent.com/83503188/178731303-a049ee13-9252-4499-b8f2-e2e6aa632819.png)


**논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

```java
public static void main(String[] args) {
	Map<PhoneNumber, String> m = new HashMap<>();
	m.put(**new PhoneNumber**(707, 867, 5309), "Jenny"); // 1

	System.out.println(**m.get(new PhoneNumber(707, 867, 5309)**)); // 2
	// 
}
```

"Jenny" 가 나와야 할 것 같지만 null을 반환한다. 여기에는 2개의 PhoneNumber 인스턴스가 사용되었다. 하나는 HashMap에 "제니"를 넣을 때 사용됐고, (논리적 동치인) 두 번째는 이를 꺼내려할 때 사용됐다.

PhoneNumber에 적절한 hashCode 메서드만 작성해주면 해결된다.

## 해결방안

### 최악의 해결방안
```java
@Override
public int hashCode() {
    return 42;
    }
```

위의 코드는 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법하다. 하지만 끔찍하게도 모든 객체에게 똑같은 값만 내어주므로  모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트처럼 동작한다.

따라서, O(1)인 해시테이블이 O(n)으로 느려진다.

### 올바른 hashCode 구현 방법

```java
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```
```java
@Override
public int hashCode() {
	// 1
	int result = name.hashCode(); // String 타입의 hashCode
	// 2
	result = 31 * result + gender.hashCode();
	result = 31 * result + age.hashCode();
	System.out.println("좋은 해시코드 구현 방법 : "+result+"  Object의 hash : "+ Objects.hash(name, gender, age));
	return result;
}
```

> 31을 곱하는 이유?
> 
> 31은 홀수이며 소수이기 때문이다.
> 
> 곱할 숫자가 짝수고 오버플로가 발생하면 -> 2를 곱하면 시프트 연산과 같은 결과를 주기 때문에 정보를 잃게 된다.

### 주의사항

1. 파생 필드는 해시코드 계산에서 제외해도 된다.
2. eqauls 비교에 사용되지 않은 필드는 반드시 제외해야 한다.
3. hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.