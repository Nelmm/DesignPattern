## 1. 빌더 패턴(Builder Pattern)이란?

**동일한 프로세스를 거쳐 다양한 구성의 인스턴스를 만드는 방법**을 말한다.

복잡한 객체를 만드는 프로세스를 독립적으로 분리할 수 있다.

**빌더 패턴은 객체를 생성할 때 생성자(Constructor)만 사용할 때 발생할 수 있는 문제를 개선하기 위해 고**안됐다.  **단계별로 복잡한 객체 생성이 가능**하며 각 중간단계에서의 상태를 유지하고 최종적으로 하위 객체들을 합친 최종 객체를 반환한다.

![builder](https://user-images.githubusercontent.com/79291114/159940108-fdb8b40f-0545-4a1f-9c9c-963bd1519b66.PNG)

- `Builder` : 빌더 인터페이스에서 인스턴스를 만드는 방법들을 추상화 함
- `ConcreateBuilderA` : Builder 인터페이스를 구현한 객체
  - `getProduct()`로 만들 객체를 최종적으로 반환
- `Director` : Builder로 만들 수 있는 다양한 방법 중 하나를 고정해 정의시켜 놓은 것
  - ex) A객체를 만들 때, 무조건 a,b,c 속성만을 가지게 Director를 만듬 :arrow_right: a,b,c 속성만을 가지는 A객체를 만들고 싶다면 해당 Director를 사용하면 됨





## 2. 팩토리 패턴과 빌더 패턴의 차이

- `팩토리 메서드 패턴` : **객체를 생성하는 패턴에 있어 추상화를 적용할 수 있는 가장 간단한 패턴**으로 단일 메서드를 사용하여 간단한 객체를 생성
  - 빌더 패턴 :arrow_right: Director가 Builder를 제어
  - 팩토리 메서드 패턴 :arrow_right: 상위 클래스가 하위 클래스를 제어
- `추상 팩토리 패턴` : 복잡한 객체 생성을 담당한다는 점에서는 빌더 패턴과 유사하나, 추상 팩토리 패턴은 **여러 팩토리 메서드를 사용하여 객체 생성하며 각 메서드들은 하위 객체를 바로 리턴**
- `빌더 패턴` : 단계별로 복잡한 객체를 생성이 가능하며, **각 중간단계에서의 상태를 유지하고 최종적으로 하위 객체들을 합친 최종 객체를 반환**
  - 즉, **빌더패턴은 사용자가 최종결과물**을 받고 **추상 팩토리 패턴은 최종 결과물에 필요한 재료들을 받게 됨**



### 2-1. 팩토리 패턴의 이슈

`팩토리 메소드 패턴`이나 `추상 팩토리 패턴`에서는 생성해야하는 **클래스에 대한 속성 값이 많을 때 아래와 같은 이슈**들이 있다.

1. 클라이언트 프로그램에서 팩토리 클래스를 호출할 때 Optional한 인자가 많아지면, **타입과 순서에 대한 관리가 어려워져 에러 발생 확률이 높아짐**
2. 경우에 따라 **필요 없는 파라미터들에 대해서 팩토리 클래스에 일일이 *NULL* 값을 넘겨줘야 함 **
3. 생성해야 하는 **sub class가 무거워지고 복잡해짐에 따라 팩토리 클래스 또한 복잡해짐**



### 2-2. 빌더 패턴에서의 해결 방법

빌더 패턴은 이러한 문제들을 해결하기 위해 `별도의 Builder 클래스`를 만들어 **필수 값에 대해서는 생성자를 통해, 선택적인 값들에 대해서는 메소드를 통해** step-by-step으로 값을 입력받은 후, **build() 메소드를 통해 최종적으로 하나의 인스턴스를 return하는 방식**이다.

크게 3가지 이슈를 해결하기위해 다음과 같은 **요구사항**을 만족해야 한다.

1. **불필요한 생성자를 만들지 않고** 객체를 만듦
2. 데이터의 **순서에 상관 없이** 객체를 만들어 냄
3. 사용자가 봤을때 **명시적**이고 쉽게 이해할 수 있어야 함





## 3. 빌더 패턴 예제

**예제 요구사항**

- 사람인 상품(`채용 광고`, `인적성 시험`)을 주문할 때 주문 데이터를 만들어야 한다.
- 주문 데이터에는 `기업 번호`, `상품명`, `상품 코드`, `가격`, `주문 날짜`, `상품 적용 날짜`, `상품 종료 날짜` 존재
- `채용 광고` 같은 경우 `상품 적용 날짜`, `종료 날짜`를 입력 받고 `인적성 시험` 같은경우 `상품 개수`를 입력받는다.

> **점층적 생성자 패턴**
> 

필요한 생성자를 오버로딩을 통한 객체 생성

```java
/**
 * 점층적 생성자 패턴
 */
public class Order {
    //기업 번호
    private Long companyNo;
    //상품명
    private String productName;
    //상품 코드
    private String productCode;
    //가격
    private Long price;
    //상품 개수
    private Long productCnt;
    //주문 날짜
    private LocalDateTime orderDate;
    //상품 적용 날짜
    private LocalDateTime startDate;
    //상품 종료 날짜
    private LocalDateTime endDate;

    public Order() {}

    //인적성 상품을 주문할 때 사용하는 생성자 -> 상품 개수를 받음
    public Order(Long companyNo, String productName, String productCode, Long price, Long productCnt, LocalDateTime orderDate) {
        this.companyNo = companyNo;
        this.productName = productName;
        this.productCode = productCode;
        this.price = price;
        this.productCnt = productCnt;
        this.orderDate = orderDate;
    }

    //채용 광고 상품을 주문할 때 사용하는 생성자 -> 상품 기간을 받음
    public Order(Long companyNo, String productName, String productCode, Long price, LocalDateTime orderDate, LocalDateTime startDate, LocalDateTime endDate) {
        this.companyNo = companyNo;
        this.productName = productName;
        this.productCode = productCode;
        this.price = price;
        this.orderDate = orderDate;
        this.startDate = startDate;
        this.endDate = endDate;
    }
}

/**
 * 메인 클래스
 */
public class Main {
    public static void main(String[] args) {
        //인적성 상품 주문 생성
        Order aptitudeOrder = new Order(
                1L,
                "인적성 검사",
                "aptitude",
                10000L,
                5L,
                LocalDateTime.now());

				//채용 상품 주문 생성
        Order recruitOrder = new Order(
                1L,
                "TOP",
                "recruit_top",
                10000L,
                LocalDateTime.now(),
                LocalDateTime.of(2022,3,2,0,0),
                LocalDateTime.of(2022,3,10,23,59)
        );
    }
}
```

위 코드처럼 `점층적 생성자 패턴`은  필요한 인자에 따라 새로운 `생성자`를 만들거나 `null`로 채워야하는 번거로움이 발생하고, 인자가 많아질수록 이 인자가 무슨 역할인지 알아보기 힘들고, 알맞지 않은 인자를 넣을 가능성도 늘어나게 된다.

또한 아래처럼 목적이 다르지만 자료형이 같은 경우 생성자는 한개만 생성할 수 밖에 없는 문제가 생긴다.

```java
public class Order {
		//Long Type
		public Order(Long price) {
        this.price = price;
    }
    //Long Type
    public Order(Long productCnt) {
        this.productCnt = productCnt;
    }
}
//컴파일 에러 발생
```

> 자바 빈 패턴
> 

```java
/**
 * 점층적 생성자 패턴
 */
public class Order {
    //기업 번호
    private Long companyNo;
    //상품명
    private String productName;
    //상품 코드
    private String productCode;
    //가격
    private Long price;
    //상품 개수
    private Long productCnt;
    //주문 날짜
    private LocalDateTime orderDate;
    //상품 적용 날짜
    private LocalDateTime startDate;
    //상품 종료 날짜
    private LocalDateTime endDate;

    //...Setter가 있다고 가정
}

public class Main {
    public static void main(String[] args) {
        Order aptitudeOrder = new Order();
        aptitudeOrder.setCompanyNo(1L);
        aptitudeOrder.setProductName("인적성 검사");
        aptitudeOrder.setProductCode("aptitude");
        aptitudeOrder.setPrice(10000L);
        aptitudeOrder.setProductCnt(5L);
        aptitudeOrder.setOrderDate(LocalDateTime.now());

        Order recruitOrder = new Order();
        recruitOrder.setCompanyNo(1L);
        recruitOrder.setProductName("TOP");
        recruitOrder.setProductCode("recruit_top");
        recruitOrder.setPrice(10000L);
        recruitOrder.setOrderDate(LocalDateTime.now());
        recruitOrder.setStartDate(LocalDateTime.of(2022,3,2,0,0));
        recruitOrder.setEndDate(LocalDateTime.of(2022,3,10,23,59));
    }
}
```

`자바빈 패턴`은 `Setter`를 이용해서 객체를 만드는 방법이다. `점층적 생성자 패턴` 의 단점인 가독성과 인자 순서로 인한 에러 발생 가능성이 보완 됐다.

하지만 함수가 인자수만큼 호출되고, 객체 호출 한번에 생성할 수 없는 단점이 있고, 각 인자마다 `Setter`가 존재하므로 `immutable(불변)` 객체를 생성할 수 없다는 문제도 있다.

> 빌더 패턴
> 

```java
/**
 * 주문 빌더 인터페이스
 */
public interface OrderBuilder {
    OrderBuilder companyNo(Long companyNo);
    OrderBuilder productName(String productName);
    OrderBuilder productCode(String productCode);
    OrderBuilder price(Long price);
    OrderBuilder productCnt(Long productCnt);
    OrderBuilder orderDate(LocalDateTime orderDate);
    OrderBuilder duration(LocalDateTime startDate, LocalDateTime endDate);
    Order getOrder();
}

/**
 * 주문 빌더 구현체
 */
public class DefaultOrderBuilder implements OrderBuilder{
    //기업 번호
    private Long companyNo;
    //상품명
    private String productName;
    //상품 코드
    private String productCode;
    //가격
    private Long price;
    //상품 개수
    private Long productCnt;
    //주문 날짜
    private LocalDateTime orderDate;
    //상품 적용 날짜
    private LocalDateTime startDate;
    //상품 종료 날짜
    private LocalDateTime endDate;

    @Override
    public OrderBuilder companyNo(Long companyNo) {
        this.companyNo = companyNo;
        return this;
    }

    @Override
    public OrderBuilder productName(String productName) {
        this.productName = productName;
        return this;
    }

    @Override
    public OrderBuilder productCode(String productCode) {
        this.productCode = productCode;
        return this;
    }

    @Override
    public OrderBuilder price(Long price) {
        this.price = price;
        return this;
    }

    @Override
    public OrderBuilder productCnt(Long productCnt) {
        this.productCnt = productCnt;
        return this;
    }

    @Override
    public OrderBuilder orderDate(LocalDateTime orderDate) {
        this.orderDate = orderDate;
        return this;
    }

    @Override
    public OrderBuilder duration(LocalDateTime startDate, LocalDateTime endDate) {
        //기간을 한번에 입력
        this.startDate = startDate;
        this.endDate = endDate;
        return this;
    }

    @Override
    public Order getOrder() {
				//입력받은 인자를 통한 주문 객체 생성
        return new Order(this.companyNo, this.productName,this.productCode,this.price, this.productCnt, this.orderDate, this.startDate, this.endDate);
    }
}

public class Order {
    //기업 번호
    private Long companyNo;
    //상품명
    private String productName;
    //상품 코드
    private String productCode;
    //가격
    private Long price;
    //상품 개수
    private Long productCnt;
    //주문 날짜
    private LocalDateTime orderDate;
    //상품 적용 날짜
    private LocalDateTime startDate;
    //상품 종료 날짜
    private LocalDateTime endDate;
		
		//* 빌더에서 객체를 생성하기 위한 생성자
    public Order(Long companyNo, String productName, String productCode, Long price, Long productCnt, LocalDateTime orderDate, LocalDateTime startDate, LocalDateTime endDate) {
        this.companyNo = companyNo;
        this.productName = productName;
        this.productCode = productCode;
        this.price = price;
        this.productCnt = productCnt;
        this.orderDate = orderDate;
        this.startDate = startDate;
        this.endDate = endDate;
    }
}

public class Main {
    public static void main(String[] args) {
        //빌더 객체 생성
        OrderBuilder aptitudeOrderBuilder = new DefaultOrderBuilder();
        //빌더를 통해 인자값 저장후 인적성 상품 주문 객체 생성
        Order aptitudeOrder = aptitudeOrderBuilder
                .companyNo(1L)
                .productName("인적성 검사")
                .productCode("aptitude")
                .price(10000L)
                .productCnt(5L)
                .orderDate(LocalDateTime.now())
                .getOrder();

        //빌더 객체 생성
        OrderBuilder recruitOrderBuilder = new DefaultOrderBuilder();
        //빌더를 통해 인자값 저장후 채용 상품 주문 객체 생성
        Order recruitOrder = recruitOrderBuilder
                .companyNo(1L)
                .productName("TOP")
                .productCode("recruit_top")
                .price(10000L)
                .orderDate(LocalDateTime.now())
                .duration(
                        LocalDateTime.of(2022,3,2,0,0),
                        LocalDateTime.of(2022,3,10,23,59)
                )
                .getOrder();
    }
}
```

빌더 패턴을 적용함으로써 다음과 같은 문제들이 해결됐다.

- 불필요한 생성자를 제거
- `Setter`가 존재하지 않기에 `immutable` 객체 생성 가능
- 가독성이 용이해짐

만약 반복적으로 같은 객체를 생성해야한다면 다음과 같이 `Director`를 사용해 코드를 좀더 심플하게 만들 수 있다.

```java
//...나머지 코드는 위와 동일

public class OrderDirector {
    private OrderBuilder orderBuilder;
    //빌더 저장
		public OrderDirector(OrderBuilder orderBuilder) {
        this.orderBuilder = orderBuilder;
    }

		//고정된 Order 객체 생성
    public Order sampleOrder() {
        return orderBuilder
                .productName("인적성 검사")
                .productCode("aptitude")
                .price(10000L)
                .productCnt(5L)
                .orderDate(LocalDateTime.now())
                .getOrder();
    }
}

public class Main {
    public static void main(String[] args) {
        //빌더 객체 생성
        OrderBuilder aptitudeOrderBuilder = new DefaultOrderBuilder();

        //Director객체에 Builder객체를 인자로 보냄
        OrderDirector orderDirector = new OrderDirector(aptitudeOrderBuilder);
        
				//Director에서 자동으로 생성된 객체 가져옴
        Order order = orderDirector.sampleOrder();
    }
}
```

여기서 기존의 자바에서 `@Builder`를 하시던 분은 위의 빌더 패턴이 생소해 보일 수 있다. 위의 빌더 패턴은 GoF에서 정의한 빌더 패턴이며, 많은 개발자들이 친숙하게 알고있는 빌더 패턴은 이펙티브 자바에서 제안한 것이다. 아래는 이펙티브 자바의 빌더 패턴 예제이다.

```java
public class Order {
    //기업 번호
    private Long companyNo;
    //상품명
    private String productName;
    //상품 코드
    private String productCode;
    //가격
    private Long price;
    //상품 개수
    private Long productCnt;
    //주문 날짜
    private LocalDateTime orderDate;
    //상품 적용 날짜
    private LocalDateTime startDate;
    //상품 종료 날짜
    private LocalDateTime endDate;

    //빌더를 생성하기위해 static으로 선언후 객체 생성
    public static Order.Builder builder() {
        return new Order.Builder();
    }

    //내부 빌더 클래스
    public static class Builder {
        //기업 번호
        private Long companyNo;
        //상품명
        private String productName;
        //상품 코드
        private String productCode;
        //가격
        private Long price;
        //상품 개수
        private Long productCnt;
        //주문 날짜
        private LocalDateTime orderDate;
        //상품 적용 날짜
        private LocalDateTime startDate;
        //상품 종료 날짜
        private LocalDateTime endDate;

        public Builder() {}

        public Order.Builder companyNo(Long companyNo) {
            this.companyNo = companyNo;
            return this;
        }
        public Order.Builder productName(String productName) {
            this.productName = productName;
            return this;
        }
        public Order.Builder productCode(String productCode) {
            this.productCode = productCode;
            return this;
        }
        public Order.Builder price(Long price) {
            this.price = price;
            return this;
        }
        public Order.Builder productCnt(Long productCnt) {
            this.productCnt = productCnt;
            return this;
        }
        public Order.Builder orderDate(LocalDateTime orderDate) {
            this.orderDate = orderDate;
            return this;
        }
        public Order.Builder startDate(LocalDateTime startDate) {
            this.startDate = startDate;
            return this;
        }
        public Order.Builder endDate(LocalDateTime endDate) {
            this.endDate = endDate;
            return this;
        }
				
				//저장된 Order.Builder 객체를 생성자 인자로 보냄
        public Order build() {
            return new Order(this);
        }
    }
    //빌더 객체를 통해 객체 생성
    public Order(Order.Builder builder) {
        this.companyNo = builder.companyNo;
        this.productName = builder.productName;
        this.productCode = builder.productCode;
        this.price = builder.price;
        this.productCnt = builder.productCnt;
        this.orderDate = builder.orderDate;
        this.startDate = builder.startDate;
        this.endDate = builder.endDate;
    }
}

public class Main {
    public static void main(String[] args) {
        Order order = Order.builder()
                .productName("인적성 검사")
                .productCode("aptitude")
                .price(10000L)
                .productCnt(5L)
                .orderDate(LocalDateTime.now())
                .build();
    }
}
```





## 4. 장점과 단점

이제까지 살펴본 빌더 패턴은 장점과 단점을 정리하면 아래와 같다.



### 2-1. 장점

1. 필요한 데이터만 설정할 수 있음
2. 유연성을 확보할 수 있음
3. 가독성을 높일 수 있음
4. 불변성을 확보할 수 있음

### 2-2. 단점

1. 약간의 성능이슈 발생
   - 빌더를 생성하는 것도 하나의 객체를 생성하는 것이고 빌더 객체가 다른 객체를 생성하는 것이기 때문에 약간의 메모리를 소비하게 됨
2. 코드 유지보수의 어려움
   - 이는 대부분의 디자인패턴들의 공통적인 단점으로 관리하는 클래스들이 많아질 수 있음
   - 빌더를 inner 클래스에 정의해도 클래스가 길어져 가독성이 떨어질 수 있음. php와 같은 언어에서는 inner 클래스를 지원해주지 않기 때문에 사용이 불가능





## 5. 마치며

빌더 패턴은 굉장히 자주 사용되는 생성 패턴 중 하나로, `Retrofit`이나 `Okhttp` 등 유명 오픈소스에서도 이 빌더 패턴을 사용하고 있다. 실무에서는 `Stream.Builder API`, `StringBuilder`, `UriComponentsBuilder` 등에 활용되는데, 위에 언급한 장점을 잘 이해하고 필요하다면, 디렉터까지 잘 활용해보면 좋을 것 같다.

