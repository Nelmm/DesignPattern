# 중재자 패턴

객체간의 종속성을 줄일 수 있는 행동관련 디자인 패턴. 객체 간의 직접 통신을 제한하고 중재자 객체를 통해서만 협력하도록 하는 패턴

## 코드

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        ControlTower controlTower = new IncheonAirportControlTower();
        Airplane asiana = new AsianaAirplane(controlTower);
        Runway runway1 = new IncheonRunWay1(controlTower);
        controlTower.registerFlight(asiana );
        controlTower.registerRunway(runway1);

        runway1.land();
        asiana.land();
    }
}
public interface ControlTower {
    public void registerRunway(Runway runway);
    public void registerFlight(Airplane airplane);
    public boolean isLandingOk();
    public void setRunwayStatus(boolean status);
}
public class IncheonAirportControlTower implements ControlTower{
    private Airplane airplane;
    private Runway runway;
    private boolean runwayStatus = false;

    @Override
    public void registerRunway(Runway runway) {
        this.runway = runway;
    }

    @Override
    public void registerFlight(Airplane airplane) {
        this.airplane = airplane;
    }

    @Override
    public boolean isLandingOk() {
        return runwayStatus;
    }

    @Override
    public void setRunwayStatus(boolean status) {
        this.runwayStatus = status;
    }
}
public interface Airplane {
    void land() throws InterruptedException;
}
public class AsianaAirplane implements Airplane{
    private final ControlTower controlTower;

    public AsianaAirplane(ControlTower controlTower) {
        this.controlTower = controlTower;
    }

    @Override
    public void land() throws InterruptedException {
        if (controlTower.isLandingOk()){
            controlTower.setRunwayStatus(false);
            System.out.println("착륙 중");
            Thread.sleep(3000);
            System.out.println("착륙 성공");
        } else {
            System.out.println("착륙 대기");
        }
    }
}
public interface Runway {
    void land();
}
public class IncheonRunWay1 implements Runway{
    private final ControlTower controlTower;

    public IncheonRunWay1(ControlTower controlTower) {
        this.controlTower = controlTower;
    }

    @Override
    public void land() {
        System.out.println("활주로 준비 완료");
        controlTower.setRunwayStatus(true);
    }
}
```

<br><br>

## 적용할 수 있는 상황

1. 클래스가 다른 클래스와 밀접하게 연결되어 있어 일부 클래스를 변경하기 어려운 경우
1. 커플링이 심해 다른 프로그램에서 재사용하기 어려운 경우

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
