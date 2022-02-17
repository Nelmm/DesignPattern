# 방문자(Visitor) 패턴

> 기존 코드를 변경하지 않고 새로운 기능 추가하는 방법을 제공하는 패턴
> 

![Untitled](https://user-images.githubusercontent.com/32676275/154392965-1d94d28a-2346-4b1a-8581-9aab9b636021.png)

> 방문자 패턴은 방문자(`Visitor`)가 방문 공간(`Element`)에 왔을 때(`Element.accept(Visitor)호출` ) 이후 행동을 방문자에게 맡기는 패턴이라 볼 수 있다.
> 

예시의 전후를 보고 비교해보자.
예제 요구사항은 다음과 같다.

- 사람인 상품 `TOP`, `FOCUS`, `PREMIUM`이 존재한다.
- 각 상품은 지역/직종 페이지의 각 상품 영역에 노출된다.
- 노출되는 곳은  `PC 사람인 웹페이지 지역/직종`,  `Phone 사람인앱 지역/직종` 이 존재한다.

이제 방문자 패턴을 적용하기전 일반적으로 코드를 작성해보자.

코드 구조는 다음과 같다.

![Untitled](https://user-images.githubusercontent.com/32676275/154392968-046088f8-0bfd-4191-9f4e-4a06ffe5c854.png)

```java
//상품 Interface
public interface Product {
		//상품 적용 메소드
    void applyProduct(Device device);
}

//Top 상품
public class TopProduct implements Product {
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Phone) {
            System.out.println("사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
        } else if (device instanceof Computer) {
            System.out.println("사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.");
        }
    }
}

//Focus 상품
public class FocusProduct implements Product{
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Phone) {
            System.out.println("사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
        } else if (device instanceof Computer) {
            System.out.println("사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.");
        }
    }
}

//Premium상품
public class PremiumProduct implements Product{
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Phone) {
            System.out.println("사람인앱 지역/직종 프리미엄 영역에 공고를 노출한다.");
        } else if (device instanceof Computer) {
            System.out.println("사람인 웹페이지 지역/직종 프리미엄 영역에 공고를 노출한다.");
        }
    }
}

//디바이스 Interface
public interface Device { }
//PC 사람인 웹페이지
public class Computer implements Device {}
//Phone 사람인 앱
public class Phone implements Device {}

public class Main {
    public static void main(String[] args) {
        Device computer = new Computer();
        Device phone = new Phone();

        Product top = new TopProduct();
        Product premium = new PremiumProduct();
        Product focus = new FocusProduct();

        top.applyProduct(computer);
        top.applyProduct(phone);

        premium.applyProduct(computer);
        premium.applyProduct(phone);

        focus.applyProduct(computer);
        focus.applyProduct(phone);
    }
}

결과
사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.
사람인앱 지역/직종 TOP 영역에 공고를 노출한다.
=============
사람인 웹페이지 지역/직종 프리미엄 영역에 공고를 노출한다.
사람인앱 지역/직종 프리미엄 영역에 공고를 노출한다.
=============
사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.
사람인앱 지역/직종 Focus 영역에 공고를 노출한다.
```

위 코드를 보면 `Product`의 `applyProduct()`를 통해 상품이 적용되고 있는 것을 확인할 수 있다.
그리고 각 상품의 `applyProduct()`에서 `Device`의 인스턴스를 분기로 처리하는 것을 볼 수 있다.

이제 이 코드의 문제점은 무엇일까?

만약 요구사항에 패드용 사람인앱이 생겨 패드 전용 지역/직종 페이지가 추가됐다고 해보자.
그럼 코드는 다음처럼 바뀔 것이다.

```java
//패드 클래스 추가
public class Pad implements Device {}

public class TopProduct implements Product {
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Phone) {
            System.out.println("사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
        } else if (device instanceof Computer) {
            System.out.println("사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.");
        } else if (device instanceof Pad) {   //패드 분기 추가
            System.out.println("패드용 사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
        }
    }
}

public class FocusProduct implements Product{
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Phone) {
            System.out.println("사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
        } else if (device instanceof Computer) {
            System.out.println("사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.");
        } else if (device instanceof Pad) {   //패드 분기 추가
            System.out.println("패드용 사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
        }
    }
}

public class PremiumProduct implements Product{
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Phone) {
            System.out.println("사람인앱 지역/직종 프리미엄 영역에 공고를 노출한다.");
        } else if (device instanceof Computer) {
            System.out.println("사람인 웹페이지 지역/직종 프리미엄 영역에 공고를 노출한다.");
        } else if (device instanceof Pad) {   //패드 분기 추가
            System.out.println("패드용 사람인앱 지역/직종 프리미엄 영역에 공고를 노출한다.");
        }
    }
}
```

위 코드를 `Pad`하나 추가한 것만으로도 `Product`를 상속한 모든 상품의 기존 코드가 변경된 것을 확인할 수 있다.

`OCP(Open-Clossed Principle)`에 위배되는 것이다. 만약 상품 개수가 더 늘어난다면 수정이 더욱더 쉽지 않을 것이다.

이제 이것을 방문자 패턴을 이용해 개선해보자.

```java
//Element에 해당
public interface Product {
    void accept(Device device);
}
public class TopProduct implements Product {
    @Override
    public void accept(Device device) {
        device.applyProduct(this);
    }
}
public class FocusProduct implements Product{
    @Override
    public void accept(Device device) {
        device.applyProduct(this);
    }
}
public class PremiumProduct implements Product{
    @Override
    public void accept(Device device) {
        device.applyProduct(this);
    }
}

//Visitor에 해당
public interface Device {
    void applyProduct(TopProduct topProduct);
    void applyProduct(PremiumProduct premiumProduct);
    void applyProduct(FocusProduct focusProduct);
}
public class Computer implements Device {
    @Override
    public void applyProduct(TopProduct topProduct) {
        System.out.println("사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(PremiumProduct premiumProduct) {
        System.out.println("사람인 웹페이지 지역/직종 프리미엄 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(FocusProduct focusProduct) {
        System.out.println("사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.");
    }
}
public class Phone implements Device {
    @Override
    public void applyProduct(TopProduct topProduct) {
        System.out.println("사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(PremiumProduct premiumProduct) {
        System.out.println("사람인앱 지역/직종 프리미엄 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(FocusProduct focusProduct) {
        System.out.println("사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
    }
}
public class Pad implements Device{
    @Override
    public void applyProduct(TopProduct topProduct) {
        System.out.println("패드용 사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(PremiumProduct premiumProduct) {
        System.out.println("패드용 사람인앱 지역/직종 프리미엄 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(FocusProduct focusProduct) {
        System.out.println("패드용 사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
    }
}

public class Main {
    public static void main(String[] args) {
        Device computer = new Computer();
        Device phone = new Phone();

        Product top = new TopProduct();
        Product premium = new PremiumProduct();
        Product focus = new FocusProduct();

        top.accept(computer);
        top.accept(phone);
        System.out.println("=============");
        premium.accept(computer);
        premium.accept(phone);
        System.out.println("=============");
        focus.accept(computer);
        focus.accept(phone);
    }
}

결과
사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.
사람인앱 지역/직종 TOP 영역에 공고를 노출한다.
=============
사람인 웹페이지 지역/직종 프리미엄 영역에 공고를 노출한다.
사람인앱 지역/직종 프리미엄 영역에 공고를 노출한다.
=============
사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.
사람인앱 지역/직종 Focus 영역에 공고를 노출한다.
```

위 코드를 보면 `Element(Product)`에는 `Visitor(Device)`를 인자로 받는 `accept()`가 정의되어있다.

그리고 각 상품에서 오버라이딩한 것을 보면 `Device`의`applyProduct()`를 호출하면서 `this(Product Instance)`인자를 보내게 된다. 

`Visitor`에는 `Device`를 상속한 클래스 인자로 오버로딩한 메소드가 정의됨으로써 각 `Device instance`별로 처리되는 구조가 된다.

프로세스를 간단히하면 다음과 같다.

> `Main` → `Element.accept(Visitor)` → `Visitor.applyProduct(Element)` → `Element Instance에 따른 오버로딩 처리`
> 

이렇게 코드를 짤 경우 이제 만약 다른 `Device`가 추가된다 하더라도 기존의 코드는 전혀 변경되지 않고 `Device`를 상속한 클래스 하나만 정의해주면 된다. 즉 OCP를 만족하게 된다.

---

### 장/단점

> 장점
> 
- 기존 코드를 수정하지 않고 확장이 가능해진다.(Device를 추가하는 것이 용이함)

> 단점
> 
- 구조 자체가 복잡해 흐름을 파악하기 어려움
- 타입마다 오버로딩 메소드를 정의해줘야함
- `Visitor` 추가에 대해서는 용이하지만 `Element` 추가, 제거에는 어려움이 있음(`Element` 추가시 모든 Visitor