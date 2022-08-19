# 가변인수는 신중히 사용하라

## 가변 인수 
```java

    static int sum(int... args) {
        int sum = 0;
        for (int arg : args)
            sum += arg;
        return sum;
    }
    
```

- 가변인수(varags) 메서드는 명시한 타입의 인수를 0개 이상 받기 가능 

```
* 가변 인수 메서드 호출 과정
1) 먼저 인수의 개수와 길이가 같은 배열을 만든다.
2) 만들어진 인수들을 이 배열에 저장한다.
3) 배열을 가변 인수 메서드에 건네준다. 
```

- 가변인수 메서드를 호출하면 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네주기 가능 

## 가변인수의 유연성
- 가변인수는 인수 개수가 정해지지 않았을대 아주 유용
- 그런데 성능에 민감한 상황이라면, 가변인수가 메모리 걸림돌 가능
### 호출 될때마다 배열 하나를 새로 할당하고 초기화하기 때문 
- 따라서 신중히 쓰자~

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... rest) {}
```

- 이 비용을 감당할 수는 없지만 가변 인수의 유연성이 필요할때 선택할 수 있는 패턴이 존재 
- 예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용  => 그렇다면 인수가 0개인 것부터 4개인 것까지 총 5개의 메서드를 다중정의하는 대안이 better 


## reference
https://gona.tistory.com/75
https://devfunny.tistory.com/624?category=895441