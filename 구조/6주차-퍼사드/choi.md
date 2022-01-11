# 퍼사드 패턴

> 어떤 **서브시스템**의 **일련의 인터페이스**에 대한 **통합된 인터페이스**를 제공합니다. 퍼사드에서 고수준 인터페이스를 정의하기 때문에 **서브시스템을 더 쉽게 사용**할 수 있습니다.
>

![Untitled](https://user-images.githubusercontent.com/32676275/147806719-7e34c6c0-859d-4cdb-8bab-c99b8dafc079.png)

특정한 기능을 구현하기 위해 써온 **많은 의존성(클래스)들을 클라이언트에 난잡하게 사용한 것**을 interface를 기능으로서 추상화해서 클라이언트에서 알아보기 쉽게 하는 패턴

밑의 예제는 출퇴근 하는 과정을 예제로 짜봤다.

```java
//회사
public class Company {
    public void turnOnNotebook() {
        System.out.println("노트북을 켜다");
    }
    public void turnOffNotebook() {
        System.out.println("노트북을 끄다");
    }
}
//집
public class Home {
    public void enterHome() {
        System.out.println("집에 들어가다");
    }
    public void getOutHome() {
        System.out.println("집에서 나오다");
    }
}
//지하철
public class Subway {
    public void takeSubway() {
        System.out.println("지하철 타다");
    }
}
//추상화 인터페이스
public interface Working {
    public void goToWork();
    public void offWork();
}
//디폴트 구현체
public class DefaultWorking implements Working{
    private final Company company;
    private final Home home;
    private final Subway subway;

    public DefaultWorking(Company company, Home home, Subway subway) {
        this.company = company;
        this.home = home;
        this.subway = subway;
    }

    @Override
    public void goToWork() {
        home.getOutHome();
        subway.takeSubway();
        company.turnOnNotebook();
    }

    @Override
    public void offWork() {
        company.turnOffNotebook();
        subway.takeSubway();
        home.enterHome();
    }
}
//클라이언트
public class Main {
    public static void main(String[] args) {
        //출근과 퇴근을 해보자.
        Company company = new Company();
        Subway subway = new Subway();
        Home home = new Home();
        //출근
        home.getOutHome();
        subway.takeSubway();
        company.turnOnNotebook();
        System.out.println("-------");
        //퇴근
        company.turnOffNotebook();
        subway.takeSubway();
        home.enterHome();
        System.out.println("-------");
        //퍼사드
        Working working = new DefaultWorking(new Company(), new Home(), new Subway());
        //출근
        working.goToWork();
        System.out.println("-------");
        //퇴근
        working.offWork();
        System.out.println("-------");
    }
}
집에서 나오다
지하철 타다
노트북을 켜다
-------
노트북을 끄다
지하철 타다
집에 들어가다
-------
집에서 나오다
지하철 타다
노트북을 켜다
-------
노트북을 끄다
지하철 타다
집에 들어가다
-------
```

위 코드를 보면 퍼사드 패턴을 적용하기전에는 클라이언트에서 직접 Company, Subway, Home 객체를 생성해 출퇴근 순서에 맞게끔 호출해줘야했다.

이럴 경우 사용자는 클래스 의존성에 대해 알아야하는것이 많고 이게 정확히 어떤 기능을 하는지 알아채기 힘들 수 있다.

이걸 퍼사드 패턴을 적용하게 되면 Working 이라는 인터페이스로 묶어 출근 기능인 goToWork()와 퇴근기능인 offWork()를 선언하면서 구현체에서는 클라이언트에 작성한 코드를 넣어 사용자는 오로지 출근과 퇴근 기능만 사용할 수 있게한다.

이로 인해 사용자는 기능의 자세한 기능을 알지 못해도 해당 기능을 쉽게 사용할 수 있는 장점이 생긴다.