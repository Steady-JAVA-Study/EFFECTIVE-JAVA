# ITEM 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 마커 인터페이스(marker interface)

> 마커 인터페이스?
>
> **아무 메서드도 담고 있지 않고**, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스
>
> ex. Serialize 인터페이스

```java
public interface Serializable {
}
```
Serialize 인터페이스는 자신을 구현한 인스턴스는 ObjectOutPutStream을 통해 쓸 수 있다고(직렬화할 수 있다고) 알려주는 역할을 한다.

실제로 무언갈 하진않고 (Serializable가) 선언되어있는지만 체크하기 때문에 마커 인터페이스라고 부른다.



## 마커 애너테이션(marker annotation)

> 마커 애너테이션?
>
> 멤버변수 없이 단순히 대상에 마킹하는 어노테이션
>
> ex. @Override

### 마커 인터페이스 vs 마커 애너테이션

1. 마커 인터페이스는 타입으로 사용할 수 있다.

- 마커 인터페이스와 마커 에너테이션 모두 클래스가 어떤 속성을 가진다는 표시를 할 수 있다.
- 마커 인터페이스는 타입으로 사용하여 컴파일타임 에 오류를 검출할 수 있다.
- 마커 에너테이션은 런타임에서야 오류를 검출할 수 있다.

```java
public class Test {

    static class Test1 {

        private final String temp;

        public Test1(String temp) {
            this.temp = temp;
        }
    }

    public static void main(String[] args) {

        Test1 p1 = new Test1("temp");
        try {
            FileOutputStream f = new FileOutputStream(new File("myObjects.txt"));
            ObjectOutputStream o = new ObjectOutputStream(f);

            o.writeObject(p1);

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

### 왜 컴파일 시점에 오류를 못잡지?

ObjectOutputStream.write API는 Serializable 인터페이스의 이점을 얻지 못한다. 왜냐하면, 파라미터 타입이 Object 이기 때문이다.


2. 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다.

마커 에너테이션은 @Retention(RetentionPolicy.TYPE) 에너테이션을 통해 클래스, 인터페이스, enum, 에너테이션에만 달 수 있는 마커를 만들 수 있다.

하지만 마커 인터페이스는 이보다 더 정밀한 제한을 둘 수 있다.


### 마커 애너테이션 사용 시점

1. 마킹하려는 곳이 클래스, 인터페이스가 아닌 경우
2. 에너테이션을 적극 활용하는 프레임워크의 일부에 마커를 다는 경우

**마커 인터페이스와 어노테이션은 각각 쓰임이 다를 뿐이다.**