# 퍼사드 패턴 예제

예제 요구사항

관리자가 기업에게 `채용홈페이지 상품`을 무료로 지급한다 했을 때 다음과 같은 프로세스가 진행된다 가정해보자.

- 주문 데이터 생성
- 서비스 적용 데이터 생성
- 채용 홈페이지 도메인 생성
- 채용 홈페이지 활성화

먼저 퍼사드 패턴을 적용하지 않고 기능을 구현해보자.

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

이제 파사드 패턴을 적용해보자.

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