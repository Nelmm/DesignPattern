## 1. 템플릿 메소드 패턴(Template method Pattern)이란?

알고리즘 구조를 서브 클래스가 확장할 수 있도록 템플릿으로 제공하는 패턴

추상 클래스는 템플릿을 제공하고 하위 클래스는 구체적인 알고리즘을 제공한다.

[![image-템플릿_메소드_01](https://user-images.githubusercontent.com/42997924/154322588-83209f4a-5841-476e-8260-5514bb8962c6.png)](https://user-images.githubusercontent.com/42997924/154322588-83209f4a-5841-476e-8260-5514bb8962c6.png)

- 구조
  - `AbstractClass` 하나와 이를 구현한 하나 이상의 `ConcreteClass`가 존재함
  - `AbstractClass`에 템플릿 메소드 역할을 하는 메소드가 있어야 함
  - 템플릿 메소드는 알고리즘의 구조를 표현한 메소드
- 데이터를 읽고 → 처리 → 출력 등의 **일련의 과정인 알고리즘 구조가 있을 때, 구조를 템플릿으로 제공** :arrow_right: 그 중 **구체적인 방법(데이터를 가져오는 방법, 처리 방법, 출력 방법 등)을 이 템플릿을 상속받는 서브 클래스가 구체적으로 구현**할 수 있도록 하는 패턴
- 상속을 사용





## 2. 코드로 알아보는 템플릿 메소드 패턴

### 2-1. 요구사항

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

이렇게함으로써 패턴 적용 전 코드처럼 **공통 로직이 중복되지 않고 서로 다른 로직만 재구현하여 사용**할 수 있다.



## 3. 장단점 및 차이점

### 3-1. 장점

- 템플릿 코드를 재사용하고 중복 코드를 줄일 수 있음
- 템플릿 코드를 변경하지 않고 상속을 받아서 구체적인 알고리즘만 변경할 수 있음 :arrow_right: `OCP`, `SRP`



### 3-2. 단점

- 리스코프 치환 원칙(`LSP`)을 위반할 수 있음
  - 예제에서 만약 PaymentProcessor를 상속받은 클래스에서 `process()`를 Override해서 예기치 못한 동작을 수행하게 만들 가능성 있음 :arrow_right: 이런 위험을 없애기 위해서 `process()` 메소드의 접근 제한자를 `final`로 해서 막을 수도 있음
  - 그러나 process() 내부에 템플릿 메서드에서 구현하고자 하는(예제의 applyService) step에 해당하는 Operation은 특정 의도가 있는 메소드 인데, 상속은 언제든지 그 의도를 깨트릴 수 있는 가능성이 있음
- 알고리즘 구조가 복잡할 수록 템플릿을 유지하기 어려워짐
  - 하위 클래스에서 알고리즘을 변경할 수 있도록 허용하는 부분이 많아질수록 템플릿 메소드 코드가 복잡해줄 수 있음

> **리스코프 치환 원칙**
> 상속 구조에서 상위 클래스 타입으로 사용하는 코드에서 (상위 타입이 아닌) 상위 타입을 상속받은 임의의 클래스 중 어떠한 것으로 바꾸더라도 코드가 의도했던 대로 동작해야한다.
> 한마디로, 상속을 받은 자식 클래스가 부모가 원래 가지고 있던 오퍼레이션 의도를 그대로 수행해야 한다.



### 3-3. 차이점

#### 3-3-1. 팩토리 메서드와의 차이점

팩토리 메서드패턴과 상속을 이용해서 특정 기능을 수행한다는 점이 매우 비슷한데 팩토리 메서드패턴은 `객체 생성`이 목적이며 템플릿메서드패턴은 `행동 수행`이 목적이라는 차이점있다.



#### 3-3-2. 템플릿 콜백 패턴과의 차이점

템플릿 콜백  패턴은 템플릿 메소드 패턴과 다르게 콜백으로 **상속(abstract) 대신 위임(interface)을 사용하는 템플릿 패턴**이다. 때문에 계산하는 로직을 추상 메서드로 분리하는 템플릿 메소드 패턴과 달리 계산 로직을 담고 있는 메소드 하나를 콜백이라는 인터페이스에 담아둔다.





## 4. 템플릿 콜백 패턴(Template-Callback Pattern)

> GoF에서는 정의하지 않은 패턴
> But. 스프링에서 많이 사용되는 패턴

콜백으로 상속(abstract) 대신 위임(interface)을 사용하는 템플릿 패턴

Callback이라는 인터페이스를 사용하여 상속 대신 **익명 내부 클래스** 또는 **람다 표현식** 을 활용할 수 있다.

[![image-템플릿_메소드_07](https://user-images.githubusercontent.com/42997924/154322595-bf172e36-bbdd-43dd-a9f6-b8880aeacc96.png)](https://user-images.githubusercontent.com/42997924/154322595-bf172e36-bbdd-43dd-a9f6-b8880aeacc96.png)

- `Callback` : 전략 패턴처럼 전략을 제공하고 해당 로직만 따로 분리할 수 있음



### 4-1. 전략 패턴과 다른 점

- 전략 패턴은 여러 개의 메소드를 가지고 있을 수 있음
- 콜백은 무조건 하나의 메소드를 가지며, 사용하는 시점에 주입하고 변경할 수 있음
  - 만약 오퍼레이터가 여러개 필요하다면 인터페이스를 여러개 만들어야 함



### 4-2. 장점

- 상속을 사용하지 않아도 됨
- 전략 패턴처럼 위임을 사용할 수 있음
- 콜백을 구현하는 방법
  1. ConcreteCallback의 새로운 구현체를 만듦
  2. 구현체를 만들지 않고  `익명 내부 클래스` 또는 `람다 표현식` 을 활용할 수 있음 :arrow_right: 좀 더 코드가 간결해짐





## 5. 마치며

개발을 하다보면 대부분의 로직과 코드는 같은데 중간에 분기 조건이나 log만 다른 경우를 종종 보곤하는데, 템플릿 메소드 패턴이나 템플릿 콜백 패턴으로 이를 개선할 수 있다.

마지막으로, 디자인 패턴을 사용할 때 다양하고 비슷한 디자인 패턴이 많기 때문에 `패턴을 구분할 때는 생김새로 구분하는 것이 아니라, 목적으로 구분해야 한다.`라는 말을 항상 명시하는 것이 좋을 것 같다.