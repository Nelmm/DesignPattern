# 템플릿 메소드 패턴 예제

예제 요구사항

- 사람인 상품 `TOP`, `Grand`을 결제 처리하는 프로세스가 필요
- 두 상품의 결제 처리 로직은 **동일**하지만 **상품을 적용할 때의 로직만 다르다 가정**

먼저 템플릿 메소드 패턴을 적용하지 않은 예제를 보자

```java
/**
 * TOP 상품 결제 처리 클래스
 */
public class TopPaymentProcessor {
    private final Long companyNo;
    public TopPaymentProcessor(Long companyNo) {
        this.companyNo = companyNo;
    }
    //TOP 상품 결제 처리
    public void process() {
        System.out.printf("=============기업 번호: %d 결제 처리===========\n", this.companyNo);
        System.out.println("주문 데이터 가져오기");
        System.out.println("결제 데이터 생성");
        System.out.println("** TOP 상품 적용 **");
        System.out.println("결제 성공 이메일 발송");
    }
}
```

```java
/**
 * 그랜드 상품 결제 처리 클래스
 */
public class GrandPaymentProcessor {
    //기업 번호
    private final Long companyNo;
    public GrandPaymentProcessor(Long companyNo) {
        this.companyNo = companyNo;
    }
    //그랜드 상품 결제 처리
    public void process() {
        System.out.printf("=============기업 번호: %d 결제 처리===========\n", this.companyNo);
        System.out.println("주문 데이터 가져오기");
        System.out.println("결제 데이터 생성");
        System.out.println("** 그랜드 상품 적용 **");
        System.out.println("결제 성공 이메일 발송");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        //TOP 상품 결제 처리
        TopPaymentProcessor topPaymentProcessor = new TopPaymentProcessor(1L);
        topPaymentProcessor.process();

        //Grand 상품 결제 처리
        GrandPaymentProcessor grandPaymentProcessor = new GrandPaymentProcessor(1L);
        grandPaymentProcessor.process();
    }
}

결과
=============기업 번호: 1 결제 처리===========
주문 데이터 가져오기
결제 데이터 생성
** TOP 상품 적용 **
결제 성공 이메일 발송
=============기업 번호: 1 결제 처리===========
주문 데이터 가져오기
결제 데이터 생성
** 그랜드 상품 적용 **
결제 성공 이메일 발송
```

`TopPaymentProcessor`, `GrandPaymentProcessor` 두 클래스의 `process()` 메소드를 비교해보면 로직이 거의 동일하고 상품을 적용하는 부분만 차이남을 볼 수 있다.

결국 많은 코드가 중복된다는 문제점이 있는 것인데 이것을 `템플릿 메소드 패턴`을 이용해 개선해보자.

![Untitled](https://user-images.githubusercontent.com/32676275/158522741-9e0a14d5-0991-4692-a039-9d93b17654aa.png)

```java
/**
 * 결제 처리 추상 클래스
 * ** AbstractClass **
 */
public abstract class PaymentProcessor {
    private final Long companyNo;
    public PaymentProcessor(Long companyNo) {
        this.companyNo = companyNo;
    }
    //결제 프로세스
    public final void process() {
        System.out.printf("=============기업 번호: %d 결제 처리===========\n", this.companyNo);
        System.out.println("주문 데이터 가져오기");
        System.out.println("결제 데이터 생성");
        //TOP, 그랜드 각 로직 실행
				applyService();
        System.out.println("결제 성공 이메일 발송");
    }
    //상품 적용 로직은 상속을 통해 구현
    protected abstract void applyService();
}
```

```java
/**
 * TOP 상품 적용 로직 구현체 클래스
 * ** ConcreteClass **
 */
public class TopProcessor extends PaymentProcessor{
    public TopProcessor(Long companyNo) {
        super(companyNo);
    }
    //TOP 상품 적용 로직 구현
    @Override
    protected void applyService() {
        System.out.println("** TOP 상품 적용 **");
    }
}
```

```java
/**
 * 그랜드 상품 적용 로직 구현체 클래스
 * ** ConcreteClass **
 */
public class GrandProcessor extends PaymentProcessor{
    public GrandProcessor(Long companyNo) {
        super(companyNo);
    }
    //그랜드 상품 적용 로직 구현
    @Override
    protected void applyService() {
        System.out.println("** 그랜드 상품 적용 **");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        //TOP 상품 결제 처리
        PaymentProcessor topProcessor = new TopProcessor(1L);
        topProcessor.process();

        //Grand 상품 결제 처리
        PaymentProcessor grandProcessor = new GrandProcessor(1L);
        grandProcessor.process();
    }
}

결과
=============기업 번호: 1 결제 처리===========
주문 데이터 가져오기
결제 데이터 생성
** TOP 상품 적용 **
결제 성공 이메일 발송
=============기업 번호: 1 결제 처리===========
주문 데이터 가져오기
결제 데이터 생성
** 그랜드 상품 적용 **
결제 성공 이메일 발송
```

`PaymentProcessor`클래스에서 `process()` 메소드에 공통 로직을 두고 상품 적용 로직을 `applyService()`메소드를 재구현으로 처리했다.

이렇게함으로써 패턴 적용전 코드처럼 공통 로직이 중복되지 않고 서로 다른 로직만 재구현하여 사용할 수 있다.