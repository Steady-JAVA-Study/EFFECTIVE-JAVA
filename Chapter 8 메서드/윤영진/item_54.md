# ITEM 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

```java
public List<String> getItem(String category){
        List<String> items = ItemMap.get(category);
        return items.isEmpty() ? null : new ArrayList<>(items);
    }
```

```java
public static void main(String[] args) {
        Item54 item = new Item54();
        List<String> items = item.getItem("과자"); // 참조 값이 null 일 수도 있음

        if(items == null){ // null 체크 로직이 반드시 들어가야함
            System.out.println("텅텅 비어있어요");
        }else{
            items.forEach(System.out::println);
        }

    }
```

null을 반환하는 코드를 작성하게 되면 해당 메서드를 사용하는 클라이언트에서 null 상황을 처리하는 코드를 추가적으로 작성해야한다.

null을 반환하는 것이 좋다는 사람들의 의견은 빈 컬렉션을 반환하는 것이 비용이 들기 때문에 null로 처리해야한다고 주장하지만

1. 성능 저하의 주범이라고 확인되지 않는 이상 이정도의 성능 차이는 신경 쓸 수준이 못된다.
2. 빈 컬렉션과 배열을 굳이 새로 할당하지 않고 만들수 있다.

## 빈 불변 컬렉션 반환

```java
private static final EMPTY_LIST = new ArrayList<>();
public static final <T> List<T> emptyList() {
        return (List<T>) EMPTY_LIST;
        }
```

또는 

```java

  public List<String> getItem(String category) {
        List<String> items = ItemMap.get(category);
        return items.isEmpty() ? Collections.emptyList() : new ArrayList<>(items);

    }
```

## 빈 배열 반환

```java
public String[] getItem(String category){
  	List<String> items = ItemMap.get(category);
  	return items.toArray(new String[0]);
}
```

toArray 메서드는 items에 원소가 있으면 items 리스트를 배열로 변환하여 반환하고, 그렇지 않으면 String[] 타입의 배열을 새로 생성해 반환한다.

여기 또한 만약 배열을 계속 생성하는것이 성능에 부담이 된다면 미리 길이 0짜리 배열을 선언하여 매번 생성한 배열을 반환하자.

```text
private static final String[] EMPTY_STRING_ARRAY = new String[0];
```

### 주의할점은 배열의 용량을 미리 할당하는 것은 추천하지 않는다(성능이 오히려 떨어진다).

```java
public String[] getItem(String category){
  	List<String> items = ItemMap.get(category);
  	return items.toArray(new String[items.size()]); // 이처럼 item.size() 를 지정하여 미리 할당하지 말라는 이야기입니다.
}
```

