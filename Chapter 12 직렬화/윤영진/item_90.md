# ITEM 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

## Proxy Pattern

대리하는 패턴? 

어떤 객체를 사용하고자 할때, 객체를 직접적으로 참조하는 것이 아닌 해당 객체를 대항하는 객체를 통해 대상 객체를 접근하는 방식

클라이언트가 어떤 일에 대한 요청을 프록시가 대신 요청하여 결과를 반환하는 패턴 

프록시 패턴을 사용함으로써 보안성이 향샹될 수 있고, 크기가 크고 메모리를 많이 사용하는 객체의 복사를 방지할 수 있다.

![image](https://user-images.githubusercontent.com/83503188/190575696-5fb703ef-bf59-4374-9f35-6abce5348d3e.png)

```java
public class Item90 {
    interface Subject {
        void request();
    }

    static class RealSubject implements Subject {

        @Override
        public void request() {
            System.out.println("Hello world");
        }
    }

    static class Proxy implements Subject {

        Subject subject;
        @Override
        public void request() {
            if(subject == null) subject = new RealSubject();
            subject.request();
        }
    }
}

```
---

## 직렬화 프록시 패턴

Serializable을 구현하는 순간 생성자 이외의 방법으로 인스턴스를 생성할 수 있게 되어 보안 문제가 생긴다. 하지만 직렬화 프록시 패턴을 사용하면 이 위험을 크게 줄일 수 있다.


```java
class Period implements Serializable {
    // 불변 가능
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    private static class SerializationProxy implements Serializable {
        private static final long serialVersionUID = 2123123123;
        private final Date start;
        private final Date end;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        // Deserialize -> Object 생성
        private Object readResolve() {
            return new Period(start, end);
        }
    }

    // Serialize -> 프록시 인스턴스 반환
    // 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // Period 자체의 역직렬화를 방지 -> 역직렬화 시도시, 에러 반환
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요해요.");
    }
}
```