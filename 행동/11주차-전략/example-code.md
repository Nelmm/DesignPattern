## 2. 전략 패턴 적용 예제

### 2-1. 요구사항

기업이 인재풀의 직무상품 결제 방식을 `네이버페이` / `신용카드 결제`를 선택적으로 할 수 있도록 설계한다.

이후에 `카카오페이`, `삼성페이` 등의 결제 수단이 추가될 것을 고려하여 확장성이 좋았으면 한다.

직무상품 1개당 가격은 만원이다.

### 2-2. 샘플 코드

<img width="682" alt="image" src="https://user-images.githubusercontent.com/42997924/163851826-6455f73a-5053-416f-bc81-cb672ea0e189.png">

```java
// Strategy 인터페이스
public interface PaymentStrategy {
    void pay(int amount);
}

// 네이버페이로 결제 (ConcreteStrategy)
@RequiredArgsConstructor
public class NaverPayStrategy implements PaymentStrategy {

    private final String naverId;
    private final String naverPw;

    @Override
    public void pay(int amount) {
        System.out.println("네이버페이로 " + amount + "원 결제했습니다.");
    }
}

// 신용카드로 결제 (ConcreteStrategy)
@RequiredArgsConstructor
public class CreditCardStrategy implements PaymentStrategy {

    private final String name;
    private final String cardNumber;
    private final String cvc;
    private final String dateOfExpiry;

    @Override
    public void pay(int amount) {
        System.out.println("신용카드로 " + amount + "원 결제했습니다.");
    }
}

// 직무 상품 Class
@RequiredArgsConstructor
public class JobItem {
    private final Integer JobIdx;
    private final String JobTitle;
}

// 직무 상품 장바구니 로직(Context)
public class ShoppingCart {

    List<JobItem> jobItemList;

    public ShoppingCart() {
        this.jobItemList = new ArrayList<>();
    }

    public void addItem(JobItem item) {
        this.jobItemList.add(item);
    }

    public void removeItem(JobItem item) {
        this.jobItemList.remove(item);
    }

    public int calculateTotal() {
        return jobItemList.size() * 10000;
    }

    // pay - PaymentStrategy 인터페이스 구현한 클래스 존재
    public void pay(PaymentStrategy paymentStrategy) {
        int amount = calculateTotal();
        paymentStrategy.pay(amount);
    }
}

// Client
public class Client {

    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();

        JobItem job1 = new JobItem(1, "서버개발");
        JobItem job2 = new JobItem(2, "안드로이드개발");

        cart.addItem(job1);
        cart.addItem(job2);

        // 네이버페이로 결제
        cart.pay(new NaverPayStrategy("myEmail@example.com", "myPwd"));
        // 신용카드로 결제
        cart.pay(new CreditCardStrategy("YoungJun Park", "1234555566667777", "999", "02/23"));
    }
}
```