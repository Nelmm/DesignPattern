# 컴포짓(Composite) 패턴

컴포짓 패턴은 **전체 계층 구조에서 그 계층 구조를 구성하는 부분적인 객체들을 클라이언트에서 동일하게 취급할 수 있게 구조를 만드는 패턴**입니다. 



## 1. 정의

컴포짓 패턴은 클라이언트는 사용하는 객체가 계층 구조 상위의 객체인지, 하위의 객체인지에 상관없이 사용할 수 있습니다. 때문에 **컴포짓 패턴은 계층 구조 즉, 트리 구조로 구성해야 한다는 제약사항**이 있습니다. 





## 2. 예시

가방에 아이템을 넣어 가격을 구하는 코드로 예시를 들어보겠습니다.



### 2-1. 컴포짓 패턴 적용 전 코드

컴포짓 패턴 적용 전 코드는 아래와 같습니다.



#### 아이템 클래스

다양한 아이템들을 표현할 수 있는 아이템 클래스입니다.

```java
public class Item {

    private String name;

    private int price;

    public Item(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public int getPrice() {
        return this.price;
    }
}
```



#### 가방 클래스

다양한 아이템들을 넣어 놓는 가방 클래스 입니다.

```java
public class Bag {

  	// 아이템들을 가지고 있음
    private List<Item> items = new ArrayList<>();

    public void add(Item item) {
        items.add(item);
    }

    public List<Item> getItems() {
        return items;
    }
}
```



#### 클라이언트

코드를 실행 시킬 수 있는 클라이언트 코드 입니다. 현재 아이템들의 가격은 클라이언트에서 구하게 됩니다.

```java
public class Client {

    public static void main(String[] args) {
      	// 아이템 생성 후
        Item doranBlade = new Item("도란검", 450);
        Item healPotion = new Item("체력 물약", 50);

      	// 가방에 넣음
        Bag bag = new Bag();
        bag.add(doranBlade);
        bag.add(healPotion);
	
      	// 클라이언트 객체를 만든 후 도란 검의 가격, 가방 안에 들어있는 아이템들의 총 가격을 구함
        Client client = new Client();
        client.printPrice(doranBlade);
        client.printPrice(bag);
    }

  	// 아이템 가격을 구하는 메서드
    private void printPrice(Item item) {
        System.out.println(item.getPrice());
    }

  	// 가방에 들어있는 아이템 총 가격을 구하는 메서드
    private void printPrice(Bag bag) {
        int sum = bag.getItems().stream().mapToInt(Item::getPrice).sum();
        System.out.println(sum);
    }

}
```



### 2-2. 컴포짓 패턴 적용 후 코드

컴포짓 패턴은 전체 계층 구조에서 그 계층 구조를 구성하는 `부분적인 객체`를 클라이언트 부분에서 동일하게 취급할 수 있게 구조를 만들어야 하기 때문에, `Bag, Item`을 `Component`를 상속받게 하여 부분적인 객체가 되어, 계층 구조를 만들 수 있습니다.

이것을 생각하면서 아래의 그림을 보게되면, `Item은 Leaf`가 되고 `여러가지 Item(Leaf)을 가지고 있는 Bag은 Composite`이 됩니다. 

![composite](https://user-images.githubusercontent.com/79291114/146661935-59ded83f-e427-4455-83be-c0fe23288793.PNG)

위의 그림에서  `Leaf`와 `Composite`은 `Componenet`를 상속받게 되기 때문에 클라이언트에서는 기존의 Item의 가격과 Bag 안의 총 가격을 구하는 메서드를 따로 구현할 필요 없이, **Component에 가격을 구하는 메서드를 만들어 놓으면 Item과 Bag을 동일한 하나의 메서드로 처리할 수 있기 때문에 동일하게 취급**할 수 있게 됩니다.



#### 컴포넌트 인터페이스

아이템과 가방이 상속받게 될 컴포넌트 인터페이스를 만듭니다.

```java
public interface Component {

  	// 상속 받은 클래스들이 구현해야하는 가격을 구하는 메서드
    int getPrice();

}
```



#### 아이템 클래스

다양한 아이템들을 표현할 수 있는 아이템 클래스입니다. 컴포넌트를 상속받았습니다.

```java
public class Item implements Component {

    private String name;

    private int price;

    public Item(String name, int price) {
        this.name = name;
        this.price = price;
    }

  	// 가격을 구하는 메서드를 Override하여 아이템의 가격을 구함
    @Override
    public int getPrice() {
        return this.price;
    }
}

```



#### 가방 클래스

다양한 아이템들을 넣어 놓는 가방 클래스 입니다. 컴포넌트를 상속받았습니다. 기존과의 큰 차이점은 **List에서 Item을 가지는 게 아니라 Component를 가지게 함으로써, 다양한 구현체들을 담을 수 있다는 것**입니다.

```java
public class Bag implements Component {

  	// 기존 처럼 Item을 가지고 있는 게 아니라, Component를 가지게 됨
    private List<Component> components = new ArrayList<>();

    public void add(Component component) {
        components.add(component);
    }

    public List<Component> getComponents() {
        return components;
    }

  	// 가격을 구하는 메서드를 Override하여 가방안에 들어있는 Component 구현체들의 총 가격을 구함
    @Override
    public int getPrice() {
        return components.stream().mapToInt(Component::getPrice).sum();
    }
}
```



#### 클라이언트

코드를 실행 시킬 수 있는 클라이언트 코드 입니다. 기존과의 차이점은 `가방 안에 들어있는 아이템들의 가격을 구하는 메서드`와 `아이템의 가격을 구하는 메서드`를 따로 구분하지 않고 `Component의 getPrice만 호출`해주면 된다는 것입니다.

```java
public class Client {

    public static void main(String[] args) {
      	// 아이템 생성 후
        Item doranBlade = new Item("도란검", 450);
        Item healPotion = new Item("체력 물약", 50);

      	// 가방에 넣음
        Bag bag = new Bag();
        bag.add(doranBlade);
        bag.add(healPotion);
	
      	// 클라이언트 객체를 만든 후 도란 검의 가격, 가방 안에 들어있는 아이템들의 총 가격을 구함
        Client client = new Client();
        client.printPrice(doranBlade);
        client.printPrice(bag);
    }

  	// 기존의 코드와 달리 Component의 getPrice만 호출해주면 됨
  	private void printPrice(Component component) {
        System.out.println(component.getPrice());
    }
}
```

위와 같이 만든 계층 구조는 아래와 같은 트리 형식이 됩니다.

<img width="815" alt="Composite-Diagram" src="https://user-images.githubusercontent.com/79291114/146667237-cb0d07bf-ec1b-4bec-8f86-1b3562185c26.png">





## 3. 장점과 단점

### 장점

- 복잡한 트리 구조를 편하게 사용할 수 있습니다.
  - 클라이언트는 Component의 getPrice 메서드만 사용하면 되기 때문

- 다형성과 재귀를 활용할 수 있습니다.
  - 하나의 `getPrice` 메서드가 구현체 마다 다르게 동작하는 `다형성`과 `Leaf 객체`를 찾기 위해 `DFS`와 같은 재귀를 활용하게 됩니다.

- 클라이언트 코드를 변경하지 않고 새로운 구현체를 추가할 수 있습니다.

컴포짓 패턴을 사용함으로써 `OCP(Open-Closed Principle)` 즉, 개방 폐쇄 원칙을 지키면서 프로그래밍을 할 수 있다는 것을 알 수 있습니다.



### 단점

- 트리를 만들야 하기 때문에 (공통된 인터페이스를 정의해야 하기 때문에) 지나치게 일반화 해야 하는 경우가 생길 수 있습니다.
  - 예를 들어, 가격이 존재하지 않는 객체가 있을 수도 있는데, 이 객체는 getPrice가 굳이 필요하지 않지만 **가방에 넣으려면 Component를 상속받아야 하기 때문에 지나친 일반화가 발생하는 경우**라고 할 수 있습니다.


컴포짓 패턴을 적용하다가 억지로 일반화해야하는 경우가 발생한다면, 해당 구조가 컴포짓 패턴으로 구현하는 게 맞는지 다시 한 번 생각해봐야 합니다.





## 4. 컴포짓 패턴을 사용하는 Swing 라이브러리

`스윙(Swing)`은 자바 언어에서 GUI의 구현하기 위해 제공되는 라이브러리입니다. 자바에서 추구하는 WORE(Wirte Once, Run Everywhere)을 구현하기 위해 `JDK 1.2` 버전부터 사용되었습니다.

```java
import javax.swing.*;

public class SwingExample {

    public static void main(String[] args) {
      	// 프레임을 만듬
        JFrame frame = new JFrame();

      	// 텍스트 필드 박스를 만들고 프레임에 추가
        JTextField textField = new JTextField();
        textField.setBounds(200, 200, 200, 40);
        frame.add(textField);

      	// 버튼을 만들고 프레임에 추가
        JButton button = new JButton("click");
        button.setBounds(200, 100, 60, 40);
        button.addActionListener(e -> textField.setText("Hello Swing"));
        frame.add(button);

      	// 프레임 크기 설정 후 보여주기
        frame.setSize(600, 400);
        frame.setLayout(null);
        frame.setVisible(true);
    }
}
```

여기서 `JFrame, JTextField, JButton`은 컴포짓 패턴으로 이루어져 있습니다. 이 3개의 객체는 전부 `Component	`라는 추상 클래스를 상속받고 있습니다.

프레임의 `add` 메서드는 아래 처럼 되어 있는데, 위 예시의 `Bag(Composite)` 같은 `Component`를 상속하는 객체들을 리스트로 가지고 있는 것을 알 수 있습니다.

```java
public Component add(Component comp) {
        addImpl(comp, null, -1);
        return comp;
}

protected void addImpl(Component comp, Object constraints, int index) {
        synchronized (getTreeLock()) {
          
          	.... 생략
              
            // component라는 리스트에 파라미터로 받은 comp를 넣습니다.
            if (index == -1) {
                component.add(comp);
            } else {
                component.add(index, comp);
            }
            
          	.... 생략
              
        }
    }
```





## 5. 컴포짓 패턴의 방식

컴포짓 패턴에서 `Composite` 클래스는 자식들을 관리하기 위한 추가적인 메서드가 필요합니다. 이러한 메서드의 설계 방식에 따라 2가지 형태의 방식으로 나눌 수 있습니다.

![composite-way](https://user-images.githubusercontent.com/79291114/147174785-16d0a493-79eb-4392-8e90-cf3475c51ca6.png)



### 안정성을 추구하는 방식

안정성을 추구하는 방식은 자식을 다루는 `add(), remove()` 와 같은 메소드들은 오직 `Composite` 만 정의되었다. 그로 인해, **Client는 Leaf와 Composite을 다르게 취급**하고 있습니다. 하지만 **Client에서 Leaf객체가 자식을 다루는 메소드를 호출할 수 없기 때문에, 타입에 대한 안정성**을 얻게 됩니다.

*먼저 예시로 들었던, `Bag`과 `Item`을 생각하면 됩니다.*



### 일관성을 추구하는 방식

일관성을 추구하는 방식은 자식을 다루는 메소드들을 `Composite`가 아닌 `Component`에 정의하는 방식입니다. 그로 인해, `Client`는 `Leaf`와 `Composite`를 일관되게 취급할 수 있습니다. 하지만 `Client`는 `Leaf` 객체가 자식을 다루는 메소드를 호출할 수 있기 때문에, 타입의 안정성을 잃게 됩니다.

`Swing`라이브러리가 일관성을 추구하는 방식으로 되어있습니다.



---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)

[컴포지트 패턴(Composite Pattern) :: 마이구미](https://mygumi.tistory.com/343)

