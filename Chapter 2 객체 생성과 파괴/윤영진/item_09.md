# ITEM 9. try-finally 보다는 try-with-resources를 사용하라

## try-finally

자바 라이브러리에는 InputStream, OutputStream 그리고 java.sql.Connection과 같이 정리(close)가 가능한 리소스가 많은데, 그런 리소스를 사용하는 클라이언트 코드가 보통 리소스 정리를 잘 안하거나 잘못하는 경우가 많다.
    
```java
public class FirstError extends RuntimeException{
}

```

```java
public class SecondError extends RuntimeException{
}

```

```java
public class MyResource implements AutoCloseable{

    public void doSomething() {
        System.out.println("Do something");
        throw new FirstError();
    }


    @Override
    public void close(){

        System.out.println("Close My Resource");
        throw new SecondError();

    }
}

```

```java
public class AppRunner {

    public static void main(String[] args) {
        MyResource myResource = new MyResource();
        try {
            myResource.doSomething();
            MyResource myResource2 = null;
            try {
                myResource2 = new MyResource();
                myResource2.doSomething();
            } finally {
                if (myResource2 != null) {
                    myResource2.close();
                }
            }
        } finally {
            myResource.close();
        }
    }
}


```
![image](https://user-images.githubusercontent.com/83503188/177958811-1f3ebaaf-6ded-48df-9e37-a8f373edbf15.png)

### 단점

1. 코드가 굉장히 지저분해진다.
2. 최초에 발생한 에러를 알 수 없다. -> 위의 로그를 보면 마지막 에러만 확인된다.

## try-with-resource

```java
public class AppRunner {

    public static void main(String[] args) {

        try (MyResource resource = new MyResource();
             MyResource resource1 = new MyResource()) {
            resource.doSomething();
            resource1.doSomething();
        }
    }
}


```
![image](https://user-images.githubusercontent.com/83503188/177960123-905bb3b4-b1e3-494e-a237-5dd42f480700.png)


코드가 굉장히 간단해진다.

모든 에러를 확인할 수 있으며 첫번째 에러를 먼저 확인할 수 있다.

