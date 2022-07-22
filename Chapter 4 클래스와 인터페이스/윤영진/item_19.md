# ITEM 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

어지간하면 상속을 하지마라.

## 상속을 금지하는 방법 

Class를 final로 선언하는 법

```java
final public class ProhibitInheritance { }
```

모든 생성자를 private or package-private로 선언하고 public static factory로 만드는 법

```java
@Getter
public class ProhibitInheritance {
    
    private int sum;
    private ProhibitInheritance() {}
    private ProhibitInheritance(int sum) {this.sum = sum}
    
    public static ProhibitInheritance getInstance(){
        return new ProhibitInheritance();
                
    }
    
    
}
```

## 상속을 하려면

@implSpec을 통해 상속할 때 필요한 내용을 서술한다.

```java

/**
 * @implSpec  method 설명
 */
public void draw(String color) {
    for (Shape shape : shapes) {
        shape.draw(color);
        }
}
```





