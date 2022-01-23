# 중재자 패턴

> 객체간의 **종속성**을 줄일 수 있는 행동관련 디자인 패턴. 객체 간의 **직접 통신**을 제한하고 중재자 객체를 통해서만 협력하도록 하는 패턴

모든 사람에게 일일히 물어보는 상황을 줄이기 위해서 **중간에 누가 한번에 알려주는 역할**만 하게끔 하는 패턴
>

[https://camo.githubusercontent.com/0f9f3d88eba374a1a81683bf42947ef8373abede7ac5aadba57b750b31ff6e23/68747470733a2f2f696d67312e6461756d63646e2e6e65742f7468756d622f523132383078302f3f73636f64653d6d746973746f72793226666e616d653d6874747073253341253246253246626c6f672e6b616b616f63646e2e6e6574253246646e253246726b434f742532466274717741336437337141253246577a4271566d416e48784b78675233573354365a6330253246696d672e706e67](https://user-images.githubusercontent.com/32676275/150677907-484dae62-cddb-4530-9a99-d5df27ff69b6.png)

---

## 어떤 경우에 사용되나

현실 세계에서 **항공기 중재**를 **관제탑**에서 관리한다 하면, 모든 비행기가 자기 주변의 비행기를 교신하면서 이동한다면, 복잡하고 어떻게 이동해야할지 많이 혼란스럽겠지만. 굳이 그럴 필요 없이 **중재자**인 **관제탑**에서 모든 책임을 지고 각 비행기에 명령을 할당하면, 훨씬 편하게 이동할 수 있을 것이다.

[https://camo.githubusercontent.com/ffcd738bb7c0f4d5963572b4789e4883a81252a60ace94ab53a5e1c6bbf48ae6/68747470733a2f2f7265666163746f72696e672e677572752f696d616765732f7061747465726e732f6469616772616d732f6d65646961746f722f70726f626c656d312d656e2e706e673f69643d3836663939303535623365363062623838333464](https://user-images.githubusercontent.com/32676275/150677978-b10ba933-512e-4cc7-b984-b2474b1b0d2d.png)

각 컴포넌트간의 의존성이 섞이는 경우가 존재하는데, 이런 경우를 방지하기 위해서는

[https://camo.githubusercontent.com/254f0b721f24349c7f03d2fa5f75ac02b7b631e9685f40ef2b8650ab05128bb9/68747470733a2f2f7265666163746f72696e672e677572752f696d616765732f7061747465726e732f6469616772616d732f6d65646961746f722f736f6c7574696f6e312d656e2e706e673f69643d6464393931613562373833306465386434336638](https://user-images.githubusercontent.com/32676275/150678017-109296d1-ee6a-464b-b397-6fef06df3ec3.png)

아예 모든 의존성을 Dialog에서 가지고 각 패턴마다 어떤 상황에 따라서 특정 행동을 할 수 있게끔 각 컴포넌트별로 이벤트를 전달하거나 값을 전달하는 방식으로 처리하면 훨씬 관리할 것들이 줄어서 편리하다.

---

## 예제 코드

해당 코드는 baeldung의 예제 코드를 참고하여 각색한 코드이다.

`팬`, `전원장치`와 `버튼`을 이용하여 `냉각 장치`를 구축한다고 할때 `버튼`을 누르면 `팬`이 켜지거나 꺼지고, `팬`을 켜기전에 `전원장치`를 켜야하는 요구사항이 존재할 것이다. 이를 아래와 같이 구현할 수 있다.

> 중재자 패턴 적용전 코드
>

```java
//팬 클래스
public class Fan {
    private final Button button;
    private final PowerSupplier powerSupplier = new PowerSupplier();
    private boolean isOn = false;

    public Fan() {
        this.button = new Button(this);
    }

    public Fan(Button button) {
        this.button = button;
    }

    public void turnOn() {
        isOn = true;
        powerSupplier.turnOn();
    }

    public void turnOff() {
        isOn = false;
        powerSupplier.turnOff();
    }

    public boolean isOn() {
        return isOn;
    }

    public Button getButton() {
        return button;
    }

    public PowerSupplier getPowerSupplier() {
        return powerSupplier;
    }
}

//버튼 클래스
public class Button {
    private Fan fan;

    public Button() {
        this.fan = new Fan(this);
    }

    public Button(Fan fan) {
        this.fan = fan;
    }

    public void press(){
        if(fan.isOn()){
            fan.turnOff();
        } else {
            fan.turnOn();
        }
    }
}

//전원 장치 클래스
public class PowerSupplier {
    public void turnOn() {
        System.out.println("파워 on");
    }

    public void turnOff() {
        System.out.println("파워 off");
    }
}

//냉각 장치 클래스
public class Circulator {
    private Fan fan;
    private Button button;
    private PowerSupplier powerSupplier;

    public Circulator() {
        this.fan = new Fan();
        this.button = fan.getButton();
        this.powerSupplier = fan.getPowerSupplier();
    }

    public Fan getFan() {
        return fan;
    }

    public Button getButton() {
        return button;
    }

    public PowerSupplier getPowerSupplier() {
        return powerSupplier;
    }
}

//클라이언트
public class Client {
    public static void main(String[] args) {
        Circulator circulator = new Circulator();
        Button button = circulator.getButton();
        button.press();
        button.press();
    }
}

//print
파워 on
파워 off
```

위 코드의 문제점은 `Fan`이 `Button`, `PowerSupplier`을 가지고 있고 `Button`이 `Fan`을 가지는 등 서로가 강하게 결합되어 있는 상태이다.

이로 인해 `Fan`이 `PowerSupplier`를 가지고 있다보니 `Circulator`에 `PowerSupplier`를 다른 객체로 추가하고자 한다면 `Button`도 `PowerSupplier`의 의존성이 생기는 문제가 발생해 확장이 어렵다. 또한, `Button`이 `Circulator`뿐만이 아닌 다른 시스템에서 사용하고자 한다면 `Fan`에 의존되어있다보니 매우 어렵다.

이제 이러한 상황을 `중재자 패턴`을 이용해 해결해보자.

> 중재자 패턴 적용후 코드
>

```java
public class Fan {
    private System system;
    private boolean isOn = false;

    public Fan(System system) {
        this.system = system;
    }

    public void turnOn() {
        system.start();
        isOn = true;
    }

    public void turnOff() {
        isOn = false;
        system.stop();
    }

    public boolean isOn() {
        return isOn;
    }
}

public class Button {
    private final System system;

    public Button(System system) {
        this.system = system;
    }

    public void press() {
        system.press();
    }
}

public class PowerSupplier {

    public void turnOn() {
        java.lang.System.out.println("파워 on");
    }

    public void turnOff() {
        java.lang.System.out.println("파워 off");
    }
}

public interface System {
    void press();
    void start();
    void stop();
}
}

public class CirculatorSystem implements System {
    private final Fan fan = new Fan(this);
    private final Button button = new Button(this);
    private final PowerSupplier powerSupplier = new PowerSupplier();

    public void press() {
        if (fan.isOn()) {
            java.lang.System.out.print("써큘레이터 ");
            fan.turnOff();
        } else {
            java.lang.System.out.print("써큘레이터 ");
            fan.turnOn();
        }
    }

    public void start() {
        powerSupplier.turnOn();
    }

    public void stop() {
        powerSupplier.turnOff();
    }

    public Button getButton() {
        return button;
    }

    public Fan getFan() {
        return fan;
    }

    public PowerSupplier getPowerSupplier() {
        return powerSupplier;
    }
}

public class Client {
    public static void main(String[] args) {
        Circulator circulator = new Circulator();
        Button button = circulator.getButton();
        button.press();
        button.press();
    }
}
//print
써큘레이터 파워 on
써큘레이터 파워 off
```

위 코드를 보면 `System`이라는 인터페이스를 `CirculatorSystem`이 상속받고 `Fan`, `Button`, `PowerSupplier`은 `System` 하나만 가지고있다. 여러 협업을 할 때는 `System` 객체를 이용해서 서로를 호출해서 사용하는 것이다.

여기서 중요한 것은 `Colleague`( `Fan`, `Button`, `PowerSupplier`)들은 오로지 `Mediator`(`System`)만을 가진다는 점이다. 이런 구조를 가질 경우 다른 `Colleague`를 추가하기도 쉬워지고 각각의 `Colleague`를 다른 `System` 구현체에서 재활용할 수 있다.

```java
public class Panel {
        private System system;
        private boolean isOn = false;

        public Panel(System system) {
            this.system = system;
        }

        public void turnOn() {
            system.start();
            isOn = true;
        }

        public void turnOff() {
            isOn = false;
            system.stop();
        }

        public boolean isOn() {
            return isOn;
        }
}

public class TvSystem implements System{
    private final Panel panel = new Panel(this);
    private final Button button = new Button(this);
    private final PowerSupplier powerSupplier = new PowerSupplier();

    public void press() {
        if (panel.isOn()) {
            java.lang.System.out.print("TV ");
            panel.turnOff();
        } else {
            java.lang.System.out.print("TV ");
            panel.turnOn();
        }
    }

    public void start() {
        powerSupplier.turnOn();
    }

    public void stop() {
        powerSupplier.turnOff();
    }

    public Button getButton() {
        return button;
    }

    public Panel getPanel() {
        return panel;
    }

    public PowerSupplier getPowerSupplier() {
        return powerSupplier;
    }
}

public class Tv {
    private TvSystem tvSystem = new TvSystem();

    public Button getButton(){
        return tvSystem.getButton();
    }
}
public class Client {
    public static void main(String[] args) {
        Tv tv = new Tv();
        tv.getButton().press();
        tv.getButton().press();
    }
}

//print
TV 파워 on
TV 파워 off
```

---

## 적용할 수 있는 상황

1. 클래스가 다른 클래스와 밀접하게 연결되어 있어 일부 클래스를 변경하기 어려운 경우
2. 커플링이 심해 클래스를 다른 프로그램에서 재사용하기 어려운 경우

---

## 자바, 스프링 사용처

> JAVA
>
- ExecutorService
- Executor

> Spring
>
- DispatcherServlet
  → 서로 간의 연결관계가 없는데, 서로를 연관지어서 관리하고 요청을 받아서 처리할 수 있는 패턴

HandlerMapping, HandlerAdapter처럼, 서로간의 의존성이 떨어져서 코드 작성이 유리해지는 방향으로 좋아졌다.

mapping을 통해서 adaptor를 찾고 ... 복잡한 방향으로 갈 수도 있었으나, 두 개를 엮어서 갈 수 있었겠지만, 그런 의존성들을 DispatcherServlet으로 몰아서 의존성을 다 한쪽으로 주고, 각 컴포넌트들은 어렵지 않게 가져가는 방향으로 설계되었다고 봐도 좋아보인다.

---

## 장단점

> 장점
>
- SRP
- OCP
- 결합도 감소
- 재사용성 증가

> 단점
>
- GOD Object로 변질
  →God Object? 많은 객체를 참조하거나 관련되지 않거나 분류가 되지 않은 메서드가 많은 객체로 안티패턴이나 Code smell의 예

---
