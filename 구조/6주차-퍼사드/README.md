# 퍼사드 패턴

어떤 **서브시스템**의 **일련의 인터페이스**에 대한 **통합된 인터페이스**를 제공함으로써 서브 시스템 의존성을 최소화하는 패턴. 

퍼사드에서 고수준 인터페이스를 정의하기 때문에 **서브시스템을 더 쉽게 사용**할 수 있다.


![Untitled](https://user-images.githubusercontent.com/32676275/147806719-7e34c6c0-859d-4cdb-8bab-c99b8dafc079.png)

특정한 기능을 구현하기 위해 써온 **많은 의존성(클래스)들을 클라이언트에 난잡하게 사용한 것**을 interface를 기능으로서 추상화해서 클라이언트에서 알아보기 쉽게 하는 패턴

<br><br>


## 1. 예제 코드
출퇴근 하는 과정을 예시로 들어 작성한 코드
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

<br><br>

## 2. 장점 / 단점

장점

- 한곳에 의존성을 몰아 넣는 효과가 있음.

단점

- 퍼사드 자체가 의존성을 가지게 됨.

<br><br>

## 3. 실제 사용 예시

### Spring

브릿지 패턴들을 대부분 퍼사드라고 봐도 관점의 차이라고 볼 수 있는것이지 다른 건 없다.

디자인 패턴으로 구현되어있는 코드는 여러가지 패턴이 섞여있다고 말할 수 있다.

- MailSender
- PlatformTransactionManager
- Spring Web MVC → webflux수도.. Servelet일 수도...

뭘쓸지는 내가 옵션으로 결정하는 방식이다. 즉, 내가 어떤 플렛폼으로 이기능을 사용할건지만 결정하면 되는 것이다.

### Laravel
```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});

```

라라벨에서 대부분의 기능들을 정적 메소드로 제공하고, 그 옵션을 Config.php 에서 작성하면 적용되고,

Memcached를 쓰던, Redis를 쓰든간에, Config에서 내 옵션을 설정만 해주면 파사드가 알아서 의존성 잡아주고, 그 기능대로 작동하는 패턴이 특히 라라벨에서 많이 채용했었다. (Cache 뿐 만 아니라 별의 별 기능에 죄다 파사드로 잡혀있다.)

하지만, 라라벨의 파사드는 클라이언트에서 의존성을 몰라도 파사드를 통해서 수행한다는 점에서 파사드와 같은 역할을 수행하지만 `다른 프레임워크의 파사드보다는 서비스 컨테이너에 대한 프록시 패턴(Proxy Pattern) 에 더 가깝습니다.` 라고 라라벨 공식홈페이지에서 설명하고 있는데 이도 알아두면 좋을 것 같습니다.

구조와 목적은 파사드로 가지고 설계를 진행했는데 일종의 프록시 역할도 수행하게 되지 않았을까 싶다.