# ITEM 53. 가변인수는 신중히 사용하라

> 가변인수?
> 
> 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.

```java
public class Item53 {
    static int min(int... args) {
        if (args.length == 0)
            throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
        int min = args[0];
        for (int i = 1; i < args.length; i++)
            if (args[i] < min)
                min = args[i];
        return min;
    }

    public static void main(String[] args) {
        System.out.println(min(1, 2, 3)); // 1
    }
}
```

위 방식에는 몇 가지 문제가 있다.

1. 가장 심각한 문제는 인수를 0개만 넣어 호출하면 (컴파일타임이 아닌) 런타임에 실패한다는 점이다.
2. 코드가 지저분하다.


```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

위의 코드처럼 매개변수를 2개 받도록 하면 된다. 즉, 첫 번째로는 평범한 매개변수를 받고, 가변인수는 두 번째로 받으면 앞의 문제가 개선된다.

## 성능에 민감한 상항이라면 가변인수 사용은 고려해보자

가변인수는 호출될 때 인수의 갯수만큼의 사이즈를 가진 배열을 만들기 때문에 성능에 민감한 경우에 걸림돌이 될 수 있다.

### 가변인수의 유연성을 활용할 수 있는 오버로딩 전략

예를 들어 메서드 호출의 95%가 인수를 N개 이하로 사용할 때, 인수가 0개인것부터 N+1개인 것까지 총 N+2개를 다중정의한다. 마지막 다중정의 메서드가 인수 N+1개 이상인 5%의 호출을 담당하게 된다.

```text
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

결과적으로 메서드 호출 중 단 5%만이 배열을 생성한다.

**EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.**
![image](https://user-images.githubusercontent.com/83503188/185100291-255a1824-dca4-4c0c-abd9-481d1aee7b6e.png)

**List.of**
![image](https://user-images.githubusercontent.com/83503188/185374465-74fa38b7-33ab-4e41-89a3-919572d735b6.png)

