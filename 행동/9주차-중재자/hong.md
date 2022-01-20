# 중재자 패턴

객체간의 종속성을 줄일 수 있는 행동관련 디자인 패턴. 객체 간의 직접 통신을 제한하고 중재자 객체를 통해서만 협력하도록 하는 패턴

## 코드
baeldung의 예제 코드를 참고하여 각색한 예제 코드이다.

팬, 전원 공급장치 와 버튼을 이용하여 냉각 장치를 구축한다고 할때 버튼을 누르면 팬이 켜지거나 꺼지고, 팬을 켜기전에 전원장치를 켜야하는 요구사항이 존재할 것이다. 이를 아래와 같이 구현할 수 있다.

```java
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

public class PowerSupplier {
    public void turnOn() {
        System.out.println("파워 on");
    }

    public void turnOff() {
        System.out.println("파워 off");
    }
}
```
이러한 요소를 가지고 써큘레이터를 만들어보자.
```java
ppublic class Circulator {
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
Fan과 Button이 강하게 결합이 되어 있다보니 fan에서 button을 만들던지 button에서 fan을 만들게 되고, Fan이 전원장치를 가지고 있다보니 냉각 시스템에 전원 장치를 다른 객체로 추가하고자 한다면 Button도 전원공급장치의 의존성이 생기는 문제가 발생해 확장이 어렵다. 또한, Button이 냉각장치뿐만이 아닌 다른 시스템에서 사용하고자 한다면 Fan에 의존되어있다보니 매우 어렵다.


이를 Mediator를 적용해 해결해보자.


### Mediator 적용후
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

Button을 재활용하지만 fan을 가지지 않는 Tv 시스템을 설계한다고 하면 아래와 같이 기존 코드 수정없이 확장할 수 있게 되었다.
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


<br><br>

## 적용할 수 있는 상황

1. 클래스가 다른 클래스와 밀접하게 연결되어 있어 일부 클래스를 변경하기 어려운 경우
1. 커플링이 심해 클래스를 다른 프로그램에서 재사용하기 어려운 경우

<br><br>

## 장단점

### 장점

1. SRP
2. OCP
3. 결합도 감소
4. 재사용성 증가

### 단점

1. GOD Object로 변질

> God Object? 많은 객체를 참조하거나 관련되지 않거나 분류가 되지 않은 메서드가 많은 객체로 안티패턴이나 Code smell의 예

<br><br>

## 다른 패턴과 비교

1. Facade : 관련이 있는 클래스들을 연결하는 점에서는 비슷하나 Facade는 단순화된 인터페이스를 정의하지만 새로운 기능을 도입하지는 않으며, 연결된 하위 시스템들은 파사드의 존재를 알지 못한다.

   Mediator의 연결된 시스템들은 중재자를 알고 있다.
