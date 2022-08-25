# ITEM 60. 정확한 답이 필요하다면 float와 double은 피하라

float과 double 타입은 과학과 공학 계산용으로 설계되었다.

이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계되었다.

따라서 정확한 결과가 필요할 때는 사용하면 안 된다.

```java
public class Item60 {
    public static void main(String[] args) throws IOException {

        System.out.println(1.03 - 0.42); // 0.6100000000000001
        System.out.println(1.00 - 9 * 0.10); // 0.09999999999999998

    }
}
```

## 해결법 

BigDecimal, int, long 을 사용하자.

