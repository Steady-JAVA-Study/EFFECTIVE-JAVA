# ITEM 18. 상속보다는 컴포지션을 사용하라

## 상속의 문제점

상위 클래스의 구현이 하위 클래스에게 노출되기 때문에 자바의 원칙 중 하나인 **캡슐화**가 깨지게 된다.

또한, 상위 클래스와 하위 클래스의 관계가 **컴파일 시점에 결정**되어 구현에 의존하기 때문에 **실행시점에 객체의 종류를 변경하는 것이 불가능**하며 **다형성과 같은 객체지향의 이점을 활요할 수 없다.**


```java
public class CustomHashSetByInheritance<E> extends HashSet<E> {

    private int addCount = 0;

    public CustomHashSetByInheritance() {
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

public class CustomHashSetTest {

    @DisplayName("더할 때마다 카운트 증가 - 상속")
    @Test
    void addAll_Inheritance() {
        CustomHashSetByInheritance<Integer> customHashSet = new CustomHashSetByInheritance<>();
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4);

        customHashSet.addAll(numbers);
        customHashSet.add(5);

        assertThat(customHashSet.getAddCount()).isNotEqualTo(5); // false
        assertThat(customHashSet.getAddCount()).isEqualTo(9); // true
    }
}
```

HashSet의 `addAll()`은 내부적으로 HashSet의 `add()`를 호출하기 때문에 재정의한 `addAll()`을 사용할 때 재정의한 `add()`가 호출되기 때문에 의도한 5가 나오지 않고 9가 나오는 잘못된 결과가 나타난다.

## 상속보다는 컴포지션(조합)을 사용하자

컴포지션이란 기존 클래스가 새로운 클래스의 구성요소로 사용되는 것을 말한다.

```java
public class CustomHashSetByComposition<E> {

    private final HashSet<E> hashSet;
    private int addCount = 0;

    public CustomHashSetByComposition(HashSet<E> hashSet) {
        this.hashSet = hashSet;
    }

    public boolean add(E e) {
        addCount++;
        return hashSet.add(e);
    }

    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return hashSet.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

public class CustomHashSetTest {

    @DisplayName("더할 때마다 카운트 증가 - 조합")
    @Test
    void addAll_Composition() {
        CustomHashSetByComposition<Integer> customHashSet = new CustomHashSetByComposition<>(
            new HashSet<>());
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4);

        customHashSet.addAll(numbers);
        customHashSet.add(5);

        assertThat(customHashSet.getAddCount()).isEqualTo(5);
    }
}
```

**상속은 캡슐화를 해칠 수 있기 때문에 is-a 관계일 때만 써야한다.**

---
# Composite Pattern

## Composite

컴포지트는 하나 이상의 유사한 객체를 구성으로 설계된 객체로 모두 유사한 기능을 나타낸다.

## Base Component 

베이스 컴포넌트는 클라이언트가 composition 내의 오브젝트들을 다루기 위해 제공되는 인터페이스를 말한다. 베이스 컴포넌트는 인터페이스 또는 추상 클래스로 정의되며 모든 오브젝트들에게 공통되는 메소드를 정의해야한다.

```java
public interface Shape {
	
    public void draw(String fillColor);
}
```

## Leaf 

composition 내 오브젝트들의 행동을 정의한다. 이는 복합체를 구성하는 데 중요한 요소이며, 베이스 컴포넌트를 구현한다. 그리고 Leaf는 다른 컴포넌트에 대해 참조를 가지면 안된다. 

```java
public class Triangle implements Shape {
 
    @Override
    public void draw(String fillColor) {
        System.out.println("Drawing Triangle with color "+fillColor);
    }
}
```
```java
public class Circle implements Shape {
 
    @Override
    public void draw(String fillColor) {
        System.out.println("Drawing Circle with color "+fillColor);
    }
}
```

## Composite

Leaf 객체들로 이루어져 있으며 베이스 컴포넌트 내 명령들을 구현한다.

Composite 객체는 Leaf 객체들을 포함하고 있으며, Base Component를 구현할 뿐만 아니라 Leaf 그룹에 대해 add와 remove를 할 수 있는 메소드들을 클라이언트에게 제공한다.

```java
public class Drawing implements Shape {
 
    //collection of Shapes
    private List<Shape> shapes = new ArrayList<Shape>();
	
    @Override
    public void draw(String fillColor) {
        for(Shape sh : shapes) {
            sh.draw(fillColor);
        }
    }
	
    //adding shape to drawing
    public void add(Shape s) {
        this.shapes.add(s);
    }
	
    //removing shape from drawing
    public void remove(Shape s) {
        shapes.remove(s);
    }
	
    //removing all the shapes
    public void clear() {
        System.out.println("Clearing all the shapes from drawing");
        this.shapes.clear();
    }
}
```

```java
public class TestCompositePattern {
 
    public static void main(String[] args) {
        Shape tri = new Triangle();
        Shape tri1 = new Triangle();
        Shape cir = new Circle();
		
        Drawing drawing = new Drawing();
        drawing.add(tri1);
        drawing.add(tri1);
        drawing.add(cir);
		
        drawing.draw("Red");
		
        List<Shape> shapes = new ArrayList<>();
        shapes.add(drawing);
        shapes.add(new Triangle());
        shapes.add(new Circle());
        
        for(Shape shape : shapes) {
            shape.draw("Green");
        }
    }
}
```


