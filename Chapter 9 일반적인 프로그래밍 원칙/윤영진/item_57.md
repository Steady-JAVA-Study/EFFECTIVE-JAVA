# ITEM 57. 지역변수의 범위를 최소화하라

**지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.**

## 지역변수의 범위를 줄이는 방법

### 1. 가장 처음에 쓰일 때 선언하기

지역변수의 범위를 줄이는 가장 강력한 기법은 가장 처음 쓰일때 선언하기이다.

미리 변수를 선언하고 나중에 할당을 하게 되면 코드가 어수선해져 가독성이 떨어진다. 또한 변수를 실제로 사용하는 시점에는 타입과 초기값이 기억나지 않을 수 있다.

### 2. 거의 모든 지역변수는 선언과 동시에 초기화

초기화에 필요한 정보가 충분치 않다면 정보가 충분해질때까지 선언을 미뤄야한다.

단, try-catch 문에서는 예외다. 변수를 초기화하는 과정에서 Checked Exception 발생 가능성이 있다면 이를 try 블록 안에서 초기화해야한다.

```java
public void test() {
    int size = 30, start = 1, end =10000, plus = 3;
    FileOutputStream fos = null;
    try{
        fos=new FileOutputStream("test.txt");
        for(int i=start;i<end; i+=plus){
        fos.write((i+", ").getBytes(StandardCharactersets.UTF_8));
        }
        }catch(Exception e) {
            System.out.println("파일을 찾을 수 없습니다.");
        }
        
}

```

```java
int memberId;
try {
    memberId = memberIdFuture.get();
} catch (InterruptException e) {
 	throw new RuntimeException();
}
```

### 3.while문 대신 for문(또는 for-each문)을 사용하기

- for문이나 for-each문을 사용한 반복문에서는 반복 변수의 범위가 반복문의 몸체, 그리고 for 키워드와 몸체 사이의 괄호 안으로 제한되기 때문에 지역변수의 범위가 줄어든다.
- 그러나 반복 변수의 값을 반복문이 종료된 뒤에도 써야 하는 상황이라면 while문을 사용하라

컬렉션이나 배열을 순회하는 권장 관용구

```java
for (Element e : c) {
    ... // e로 무언가를 한다.
}
```

- 반복자를 사용해야 하는 상황이면 for-each문 대신 전통적인 for문을 쓰는 것이 낫다

```java
List<String> like = Arrays.asList("yoon1", "yoon2", "yoon3");
    for (Iterator<String> i = like.iterator(); i.hasNext();) {
      System.out.print(i.next());
    }
```

### 4. 매서드를 작게 유지하고 한 가지 기능에 집중하라

한 메서드에서 여러 기능을 처리한다면 각 기능을 수행하는 코드가 임의의 지역변수에 접근함으로써 부수효과가 발생할 수 있다.

따라서 메서드를 기능별로 쪼개서 구현할 것을 권장한다.

```java
// 고객시점
public void order(생략) {
    // 1. 카드정보 확인
    // 2. 업체 정보 확인
    // 3. 업체 주문가능여부 확인
    // 4. 주문처리
    // 5. 결제 처리
    // 6. 주문 완료 push 발송
        
}

public boolean isAvailablePayment() {...}
public boolean isOrderAble() {...}
public DeliveryOrderDto Deliveryorder() {...}
public boolean requestPayment() {...}
public boolean push() {...}
```


