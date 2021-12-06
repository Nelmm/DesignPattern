# 팩토리 메소드 패턴

> 객체를 생성하기 위한 인터페이스를 정의하는 과정에서 
어떤 클래스의 인스턴스를 만들지는 서브클래스에서 결정
즉, 클래스의 인스턴스를 만드는 일을 서브클래스에게 맡기는 것
> 

**핵심은 공통적인 로직을 부모 Interface(또는 abstract class)의 구현체로 구현하고 변경될 수 있는 부분들을 서브 클래스로 만든다.**

### 팩토리 메소드 패턴 적용할 때 스텝

1. 공통 로직이 있고, 클래스마다 객체를 분리해야하는지 확인
2. interface or abstract에서 공통 로직 작성
3. 로직에서 사용될 서로 달라질 수 있는 객체 부분(createShip)부분을 추상 메소드를 통해 서브 클래스에게 객체를 생성하게 끔 함.

### **Default method (번외 Java8)**

Java8의 새로운 기능으로 Default method가 추가되었다. 
Default method는 **인터페이스안에 구현체 메소드를 작성할 수 있고 오버라이딩으로 재정의 하여 사용 가능**

### **Interface Private Method(번외 Java9)**

Java9의 새로운 기능으로 Interface Private Method 정의가 가능해졌다.
Interface Private Method를 사용해 Interface안에 default method처럼 구현체를 작성할 수 있다.
Private Method는 static, non-static 둘다 사용가능하다.
또한 Private Method는 Default Method, Static Method에서만 사용 가능하다
⇒ Private → Default Method, Static Private → Static Method
하지만 Default Method처럼 하위 클래스에서 상속은 불가능하다.
⇒ Default Method, Static Method의 로직을 분리하기 위해 사용

> **Private Method 예제**
> 

```java
//default, static 모두 private static 사용 가능
//private는 default에서만 사용 가능
public interface TestInterface {
    default int ab() {
        return a() + aa();
    }
    static int asds() {
        return 2 + a();
    }
    private static int a() {
        return 1;
    }
    private int aa() {
        return 2;
    }
}
```

> 팩토리 메소드 패턴 예제
> 

```java
public class Client {

    public static void main(String[] args) {
        Client client = new Client();
        client.print(new APizzaFactory());
        client.print(new BPizzaFactory());
    }

    private void print(PizzaFactory pizzaFactory) {
        pizzaFactory.requestOrder();
    }

}

public interface PizzaFactory {
    default void requestOrder() {
        prepare();
        Pizza pizza = createPizza();
        pizza.makePizza();
        delivery(pizza.getPizza());
        complete();
    }
    default void prepare(){
        System.out.println("피자 만들 준비중 ");
    }
    default void delivery(String pizza) {
        System.out.println(pizza + " 배달중");
    }
    default void complete() {
        System.out.println("피자 배달 완료");
    }

    Pizza createPizza();    //서로 다른 클래스
}

public class APizzaFactory implements PizzaFactory {
    @Override
    public Pizza createPizza() {
        return new APizza();
    }
}

public class BPizzaFactory implements PizzaFactory{
    @Override
    public Pizza createPizza() {
        return new BPizza();
    }
}

public interface Pizza {
    void makePizza();
    String getPizza();
}

@Getter
public class APizza implements Pizza{
    private String name;
    @Override
    public void makePizza() {
        System.out.println("A피자 만드는중");
        name = "A피자";
    }

    @Override
    public String getPizza() {
        return name;
    }
}

@Getter
public class BPizza implements Pizza{
    private String name;
    @Override
    public void makePizza() {
        System.out.println("B피자 만드는중");
        name = "B피자";
    }

    @Override
    public String getPizza() {
        return name;
    }
}
```

> 다이어그램
> 

![Untitled](https://user-images.githubusercontent.com/32676275/144534950-ef61780b-dceb-4ad1-b0e7-8b8a9e2b1eb0.png)

**팩토리 메소드 패턴 장점**

확장에 열려있고 변경에 닫혀있는 객체 지향
기존 비즈니스 로직을 건드리지 않고  객체적인 확장이 가능.
→ 커플링이 느슨해지기 때문에 가능
→ **기존 코드를 변경하지 않으면서 새로운 기능을 확장**

**팩토리 메소드 패턴 단점**

각자의 역할마다 클래스로 나눠지기 때문에 어쩔 수 없이 클래스의 개수가 늘어남