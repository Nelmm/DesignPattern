# [생성 패턴] 빌더 패턴 (Builder Pattern)

빌더 패턴은 생성과 관련된 디자인 패턴으로, 동일한 프로세스를 거쳐 다양한 구성의 인스턴스를 만드는 패턴 을 말합니다.



## 1. 정의(Definition)

-   GoF 디자인 패턴 중 생성 패턴에 해당한다.
-   빌더 패턴은 복잡한 객체를 생성하는 클래스와 표현하는 클래스를 분리하여, 동일한 절차에서도 서로 다른 표현 클래스 생성 방법을 제공한다.
-   생성해야하는 객체가 Optional한 속성을 많이 가질 때 더 좋다.





## 2. 의의

빌더 패턴은 객체를 생성할 때 **생성자(Constructor)만 사용할 때 발생할 수 있는 문제를 개선**하기 위해 고안됐습니다.



### 2-1. 팩토리 패턴과의 비교

- `팩토리 메서드 패턴` : 객체를 생성하는 패턴에 있어 추상화를 적용할 수 있는 가작 간단한 패턴으로 단일 메서드를 사용하여 간단한 객체를 생성

  빌더패턴은 Director가 Builder를 제어하지만, 팩토리 메서드 패턴은 상위 클래스가 하위 클래스를 제어합니다.

- `추상 팩토리 패턴` : 복잡한 객체 생성을 담당한다는 점에서는 빌더 패턴과 유사하나 여러 팩토리 메서드를 사용하여 객체 생성하며 각 메서드들은 하위 객체를 바로 리턴합니다.

- `빌더 패턴` : 단계별로 복잡한 객체를 생성이 가능하며 각 중간단계에서의 상태를 유지하고 최종적으로 하위 객체들을 합친 최종 객체를 반환합니다.

  한마디로 빌더패턴은 사용자가 최종결과물을 받고 추상 팩토리 패턴은 최종 결과물에 필요한 재료들을 받게 됩니다.

팩토리 메소드 패턴이나 추상 팩토리 패턴에서는 생성해야하는 클래스에 대한 속성 값이 많을 때 아래와 같은 이슈들이 있습니다.

1. 클라이언트 프로그램에서 팩토리 클래스를 호출할 때 Optional한 인자가 많아지면, **타입과 순서에 대한 관리가 어려워져 에러 발생 확률이 높아짐**
2. 경우에 따라 **필요 없는 파라미터들에 대해서 팩토리 클래스에 일일이 _NULL_ 값을 넘겨줘야함  **
3. 생성해야 하는 **sub class가 무거워지고 복잡해짐에 따라 팩토리 클래스 또한 복잡해짐**



### 2-2. 빌더 패턴에서의 해결 방법

빌더 패턴은 이러한 문제들을 해결하기 위해 별도의 Builder 클래스를 만들어 **필수 값에 대해서는 생성자를 통해, 선택적인 값들에 대해서는 메소드를 통해** step-by-step으로 값을 입력받은 후에 **build() 메소드를 통해 최종적으로 하나의 인스턴스를 return하는 방식**입니다.

크게 3가지 이슈를 해결하기위해 다음과 같은 **요구사항**을 만족해야 합니다.

1. **불필요한 생성자를 만들지 않고** 객체를 만듦
2. 데이터의 **순서에 상관 없이** 객체를 만들어 냄
3. 사용자가 봤을때 **명시적**이고 쉽게 이해할 수 있어야 함





## 3. 예시

여행 계획을 세우는 앱을 개발한다고 할때, 다음과 같은 요구사항이 있다고 가정합니다.

- `요구사항 1` : 여행 계획 항목은 이렇게 해주세요.
  - 여행 제목
  - 여행 출발 일
  - 몇박 며칠 동안 어디서 머물지
  - n일차에 할일을 여러가지 기록

위의 `요구 사항 1`을 만족하기 위해 아래와 같은 도메인을 구성할 수 있습니다.

```java
/**
 * 여행 계획
 */
public class TourPlan {
    private String title; // 여행 제목
    private LocalDate startDate; // 출발 일
    private int nights; // 몇 박
    private int days; // 며칠
    private String whereToStay; // 어디서 머물지
    private List<DetailPlan> plans; // n일차 할 일
}

/**
 * n일차 할 일
 */
public class DetailPlan {
    private int day; // n일차
    private String plan; // 할 일
}
```



### 3-1. 여러 가지 생성 패턴

하지만 만약 아래와 같이 필수적인 정보와 선택적인 정보로 Optional한 속성들이 생겼을 때 어떻게 구현하면 될까요?

- `요구사항 2` : 당일 치기 계획도 필요해요
  - 당일 치기는 n박m일이 필요 없음
  - 어디서 머물지도 필요없음



#### 점층적 생성자 패턴

이런 경우에는 `점층적 생성자 패턴`을 적용하면 생성자 오버로딩을 통해 구현할 수 있습니다.

```java
/**
 * 기본 생성자 (필수)
 */
public TourPlan() {
}

/**
 * 일반적인 여행 계획 생성자
 *
 * @param title 여행 제목
 * @param startDate 출발 일
 * @param nights n박
 * @param days m일
 * @param whereToStay 머물 장소
 * @param plans n일차 할 일
 */
public TourPlan(String title, LocalDate startDate, int nights, int days,
    String whereToStay, List<DetailPlan> plans) {
    this.title = title;
    this.nights = nights;
    this.days = days;
    this.startDate = startDate;
    this.whereToStay = whereToStay;
    this.plans = plans;
}

/**
 * 당일치기 여행 계획 생성자
 *
 * @param title 여행 제목
 * @param startDate 출발 일
 * @param plans 1일차 할 일
 */
public TourPlan(String title, LocalDate startDate, List<DetailPlan> plans) {
    this.title = title;
    this.startDate = startDate;
    this.plans = plans;
}
```

위와 같이 `점층적 생성자 패턴`으로 구현하면 Optional한 인자에 따라 새로운 생성자를 만들거나, Null 값으로 채워야하는 번거로움이 발생합니다.

또한, 아래와 같이 파라미터 갯수와 파라미터 타입이 동일한 생성자는 한개만 정의가 가능하기 때문에 서로 다른 목적으로 사용하고 싶을때 사용할 수 없습니다.

```java
public static void main(String[] args){
    //Constructor
    Person onlyFirstName = new Person("길동");
    Person onlyName = new Person("길동","홍");
    Person onlyLastName = new Person("홍"); //불가능
}
```

----

물론 Lombok의 `@AllArgsConstructor` 어노테이션을 활용하면 코드가 길어지는 문제는 해결할 수 있지만, 여전히 **인자가 많은 경우 타입과 순서로 발생할 수 있는 에러 가능성이 존재**합니다.

``` java
// 순서를 파악이 어렵고, 가독성이 떨어짐
new TourPlan("여행 계획", LocalDate.of(2021,12, 24), 3, 4, "호텔",
    Collections.singletonList(new DetailPlan(1, "체크인")));
    
// 생성자를 만들지 않고 당일치기 객체를 생성하면 불필요한 Null을 채워야함
new TourPlan("여행 계획", LocalDate.of(2021,12, 24), null, null, null,
    Collections.singletonList(new DetailPlan(1, "놀고 돌아오기")));
```
<details><summary>사실 IntelliJ는 똑똑하고 친절해서 hint가 뜨긴한다.</summary>

![study-designPattern-3th-builder-01](https://user-images.githubusercontent.com/42997924/145449696-6792e921-8232-4cd4-9864-26f468e704d1.png)

친절한 IDE..

</details>   




#### 자바 빈(Bean) 패턴

점층적 생성자 패턴의 단점을 보완한 setter 메소드를 사용한 자바 빈(Bean) 패턴도 있습니다.

```java
TourPlan tourPlan = new TourPlan();
tourPlan.setTitle("칸쿤 여행");
tourPlan.setNights(2);
tourPlan.setDays(3);
tourPlan.setStartDate(LocalDate.of(2021, 12, 24));
tourPlan.setWhereToStay("리조트");
tourPlan.addPlan(1, "체크인 이후 짐풀기");
tourPlan.addPlan(1, "저녁 식사");
tourPlan.addPlan(2, "조식 부페에서 식사");
tourPlan.addPlan(2, "해변가 산책");
tourPlan.addPlan(2, "점심은 수영장 근처 음식점에서 먹기");
tourPlan.addPlan(2, "리조트 수영장에서 놀기");
tourPlan.addPlan(2, "저녁은 BBQ 식당에서 스테이크");
tourPlan.addPlan(3, "조식 부페에서 식사");
tourPlan.addPlan(3, "체크아웃");
```

가독성도 좋아지고, 순서에도 자유롭기 때문에 에러 발생 가능성도 줄어듭니다. 하지만, 아래와 같이 해결하지 못한 문제점들이 있습니다.

1. 함수 호출이 인자만큼 이루어지고, 객체 호출 한번에 생성할 수 없음
2. `immutable 객체`를 생성할 수 없다. (setter로 값 변경 가능)
   - 쓰레드간 공유 가능한 객체 `일관성(consistency)`이 일시적으로 깨질 수 있음



### 3-2. 빌더 패턴

빌더 패턴은 **생성자 패턴과 자바 빈 패턴의 장점을 결합하여 객체 생성과 관련된 문제를 해결**했습니다.

-   필요한 객체를 직접 생성하는 대신, 먼저 생성자의 필수 인자들을 전부 전달받는 빌더 객체를 만듦
-   선택 인자는 가독성이 좋은 코드로 인자를 넘길 수 있음
-   setter가 없기 때문에 객체 일관성을 유지하여 불변 객체로 생성할 수 있음

기본적인 Class Diagram은 다음과 같습니다.

![study-designPattern-3th-builder-02](https://user-images.githubusercontent.com/42997924/145450029-f57c98d3-fff5-4176-878e-b08ee952d045.png)

여기서 Director에 대한 설명은 아래에서 진행하겠습니다.



#### TourPlanBuilder

우선 먼저 기본적인 인자들을 받는 Interface인 `TourPlanBuilder`를 만들어줍니다.

```java
public interface TourPlanBuilder {

    TourPlanBuilder nightsAndDays(int nights, int days);

    TourPlanBuilder title(String title);

    TourPlanBuilder startDate(LocalDate localDate);

    TourPlanBuilder whereToStay(String whereToStay);

    TourPlanBuilder addPlan(int day, String plan);

    TourPlan getPlan();

}
```



#### DefaultTourBuilder(ConcreteBuilder)

TourPlanBuilder를 구현하는 `DefaultTourBuilder`는 아래와 같습니다.

지금은 기본적인 인자만 받게 되어있지만, 상황에 따라서 Optional한 인자를 받는 구현체를 만들어 사용할 수도 있습니다.

```java
public class DefaultTourBuilder implements TourPlanBuilder {

    private String title;

    private int nights;

    private int days;

    private LocalDate startDate;

    private String whereToStay;

    private List<DetailPlan> plans;

    @Override
    public TourPlanBuilder nightsAndDays(int nights, int days) {
        this.nights = nights;
        this.days = days;
        return this;
    }

    @Override
    public TourPlanBuilder title(String title) {
        this.title = title;
        return this;
    }

    @Override
    public TourPlanBuilder startDate(LocalDate startDate) {
        this.startDate = startDate;
        return this;
    }

    @Override
    public TourPlanBuilder whereToStay(String whereToStay) {
        this.whereToStay = whereToStay;
        return this;
    }

    @Override
    public TourPlanBuilder addPlan(int day, String plan) {
        if (this.plans == null) {
            this.plans = new ArrayList<>();
        }

        this.plans.add(new DetailPlan(day, plan));
        return this;
    }

    @Override
    public TourPlan getPlan() {
        return new TourPlan(title, startDate, days, nights, whereToStay, plans);
    }
}
```

이제 아래와 같은 방법으로 TourPlan 객체를 생성할 수 있습니다.

```java
return tourPlanBuilder.title("칸쿤 여행")
        .nightsAndDays(2, 3)
        .startDate(LocalDate.of(2020, 12, 9))
        .whereToStay("리조트")
        .addPlan(0, "체크인하고 짐 풀기")
        .addPlan(0, "저녁 식사")
        .getPlan();
```

확실히 다른 패턴으로 생성하는 것보다 **체이닝을 통한 좋은 가독성**과 **Setter를 사용하지 않아 불변 객체**로 만들 수 있으며, **데이터의 순서와 상관없이 객체를 생성**할 수 있습니다.



#### Director

![study-designPattern-3th-builder-03](https://user-images.githubusercontent.com/42997924/145450185-0479a78a-fd31-4bc5-a5ce-941a797402ee.png)

Builder로 객체 생성하는 방법을 미리 정의한 `Director`를 이용하면 클라이언트 코드를 더 짧게 만들 수 있습니다.

```java
public class TourDirector {

    private TourPlanBuilder tourPlanBuilder;

    public TourDirector(TourPlanBuilder tourPlanBuilder) {
        this.tourPlanBuilder = tourPlanBuilder;
    }

    public TourPlan cancunTrip() {
        return tourPlanBuilder.title("칸쿤 여행")
                .nightsAndDays(2, 3)
                .startDate(LocalDate.of(2020, 12, 9))
                .whereToStay("리조트")
                .addPlan(0, "체크인하고 짐 풀기")
                .addPlan(0, "저녁 식사")
                .getPlan();
    }

    public TourPlan longBeachTrip() {
        return tourPlanBuilder.title("롱비치")
                .startDate(LocalDate.of(2021, 7, 15))
                .getPlan();
    }
}
```

```java
public static void main(String[] args) {
    TourDirector director = new TourDirector(new DefaultTourBuilder());
    TourPlan tourPlan = director.cancunTrip();
}
```

위와 같이 Director를 이용해 여러가지 객체 생성 메서드를 만들게 되면, `동일한 생성 절차`에서 `다른 객체`를 만들 수 있습니다.





## 4. 장점과 단점

이제까지 살펴본 빌더 패턴은 장점과 단점을 정리하면 아래와 같습니다.



### 4-1. 장점

1. 필요한 데이터만 설정할 수 있음  
2. 유연성을 확보할 수 있음 
3. 가독성을 높일 수 있음  
4. 불변성을 확보할 수 있음



### 4-2. 단점

1. 약간의 성능이슈 발생
   - 빌더를 생성하는 것도 하나의 객체를 생성하는 것이고 빌더 객체가 다른 객체를 생성하는 것이기 때문에 약간의 메모리를 소비하게 됨
2. 코드 유지보수의 어려움
   - 이는 대부분의 디자인패턴들의 공통적인 단점으로 관리하는 클래스들이 많아질 수 있음
   - 빌더를 inner 클래스에 정의해도 클래스가 길어져 가독성이 떨어질 수 있음. php와 같은 언어에서는 inner 클래스를 지원해주지 않기 때문에 사용이 불가능.



## 4. 마치며

빌더 패턴은 굉장히 자주 사용되는 생성 패턴 중 하나로, `Retrofit`이나 `Okhttp` 등 유명 오픈소스에서도 이 빌더 패턴을 사용하고 있습니다. 실무에서는 `Stream.Builder API`, `StringBuilder`, `UriComponentsBuilder` 등에 활용됩니다.

위에 언급한 장점을 잘 이해하고 필요하다면, 디렉터까지 잘 활용해보면 좋을 것 같습니다.



---

참고 : [https://dev-youngjun.tistory.com/197](https://dev-youngjun.tistory.com/197)

