## 1. 파사드 패턴(Facade Pattern) 이란?

파사드 패턴은 **서브시스템**의 **일련의 인터페이스**에 대한 **통합된 인터페이스**를 제공함으로써 서브 시스템 의존성을 최소화하는 패턴이다.

퍼사드에서 고수준 인터페이스를 정의하기 때문에 **서브시스템을 더 쉽게 사용**할 수 있다.

![facade](https://user-images.githubusercontent.com/79291114/162228866-ddae783b-b466-4e30-a7dc-2250508aaa04.PNG)

특정한 기능을 구현하기 위해 써온 **많은 의존성(클래스)들을 클라이언트에 난잡하게 사용한 것**을 interface를 기능으로서 추상화해서 클라이언트에서 알아보기 쉽게 하는 패턴이다.





## 2. 코드로 알아보는 파사드 패턴

### 2-1. 요구사항

관리자가 기업에게 `채용홈페이지 상품`을 무료로 지급한다 했을 때 다음과 같은 프로세스가 진행된다 가정해보자.

- 주문 데이터 생성
- 서비스 적용 데이터 생성
- 채용 홈페이지 도메인 생성
- 채용 홈페이지 활성화



### 2-2. 파사드 패턴 적용 전

```java
/**
 * 주문 서비스 클래스
 */
public class OrderService {
    public void createOrder() {
        System.out.println("주문 데이터 생성");
    }
}

/**
 * 서비스 적용 클래스
 */
public class ServiceApplyService {
    //서비스 적용 데이터 생성
    public void applyService() {
        System.out.println("서비스 적용 데이터 생성");
    }
}

/**
 * 채용 홈페이지 서비스 클래스
 */
public class RecruitinService {
    //채용 홈페이지 API
    private final RecruitinAPI recruitinAPI = new RecruitinAPI();

    //도메인 생성
    public void createDomain() {
        recruitinAPI.createDomainApi();
    }

    //도메인 활성화
    public void activateDomain() {
        recruitinAPI.activateDomainApi();
    }

    /**
     * 채용 홈페이지 API
     */
    private static class RecruitinAPI {
        //도메인 생성 API
        public void createDomainApi() {
            System.out.println("채용홈페이지 도메인 생성 API 호출");
        }

        //도메인 활성화 API
        public void activateDomainApi() {
            System.out.println("채용홈페이지 도메인 활성화 API 호출");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        //주문 서비스 
        OrderService orderService = new OrderService();
        //서비스 적용 서비스
        ServiceApplyService serviceApplyService = new ServiceApplyService();
        //채용홈페이지 서비스
        RecruitinService recruitinService = new RecruitinService();

        //주문 생성
        orderService.createOrder();
        //서비스 적용
        serviceApplyService.applyService();
        //도메인 생성
        recruitinService.createDomain();
        //도메인 활성화
        recruitinService.activateDomain();
    }
}

결과
주문 데이터 생성
서비스 적용 데이터 생성
채용홈페이지 도메인 생성 API 호출
채용홈페이지 도메인 활성화 API 호출
```

위의 코드를 보면 관리자가 `채용홈페이지`를 지급하기 위해 필요한 프로세스를 이해하고 구현해야한다. 예제 코드는 비교적 간단해서 바로 이해할 수 있지만  만약 더 복잡한 코드라면 다른 개발자가 코드를 통해 기능을 유추하기 힘들 수 있다.



### 2-3. 파사드 패턴 적용 후

```java
//...기존 클래스는 위와 동일

/**
 * 채용 홈페이지 적용을 위한 인터페이스
 */
public interface ApplyRecruitin {
    //관리자 채용 홈페이지 적용할 메소드
    public void applyRecruitinForAdmin();
}

/**
 * 채용 홈페이지 적용할 통합 인터페이스 구현체
 */
public class DefaultApplyRecruitin implements ApplyRecruitin{
    //주문 서비스
    private OrderService orderService;
    //서비스 적용 서비스
    private ServiceApplyService serviceApplyService;
    //채용홈페이지 서비스 
    private RecruitinService recruitinService;

    
    //외부에서 필요한 서비스들을 주입받는다.
    public DefaultApplyRecruitin(OrderService orderService, ServiceApplyService serviceApplyService, RecruitinService recruitinService) {
        this.orderService = orderService;
        this.serviceApplyService = serviceApplyService;
        this.recruitinService = recruitinService;
    }
    
    //관리자 채용홈페이지 적용 기능 제공
    @Override
    public void applyRecruitinForAdmin() {
        orderService.createOrder();
        serviceApplyService.applyService();
        recruitinService.createDomain();
        recruitinService.activateDomain();
    }
}

public class Main {
    public static void main(String[] args) {
				//관리자 채용홈페이지 적용 인터페이스
        DefaultApplyRecruitin defaultApplyRecruitin = new DefaultApplyRecruitin(new OrderService(), new ServiceApplyService(), new RecruitinService());
        //관리자 지급에 필요한 모든 서브 기능들을 통합하여 하나의 메소드로 호출
				defaultApplyRecruitin.applyRecruitinForAdmin();
    }
}
```

위 코드를 보면 `ApplyRecruitin Interface`  구현체를 구현함으로써 관리자 채용홈페이지 지급을 위해 필요한 프로세스들을 모두 통합하여 클라이언트(Main)에서는 내부 로직을 모르고 `applyRecruitinForAdmin()` 만 호출함으로써 기능을 사용할 수 있다.





## 3. 장점 / 단점

### 3-1. 장점

- 한곳에 의존성을 몰아 넣는 효과가 있음.

### 3-2. 단점

- 파사드 자체가 의존성을 가지게 됨.





## 4. 마치며

파사드 패턴은 간단해 보여서 그냥 지나칠 수 있다. 하지만 한 곳에 의존성을 몰아 넣고 네이밍을 신경쓴다면 협업을 하거나 다른 사람이 자신의 코드를 볼 때 가독성있는 코드로 만들어 줄 수 있는 패턴이라고 생각된다.