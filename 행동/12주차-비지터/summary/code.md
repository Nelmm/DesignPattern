## 코드
### 요구사항
- 사람인 상품 `TOP`, `FOCUS` 이 존재한다.
- 각 상품은 지역/직종 페이지의 각 상품 영역에 노출된다.
- 노출되는 곳은  `PC 사람인 웹페이지 지역/직종`,  `Mobile 사람인앱 지역/직종` 이 존재하며 디바이스 별로 상품 노출 로직이 상이하다.

### 적용전 코드
이제 방문자 패턴을 적용하기전 일반적으로 코드를 작성해보자.
```java
//디바이스 Inteface
public interface Device {}

//Pc일경우의 공통 로직 정의 가능
public class Pc implements Device{}

//모바일 일경우의 공통 로직 정의 가능
public class Mobile implements Device{}


//상품 Interface
public interface Product {
    void applyProduct(Device device);  //상품 적용 메소드
}

//Focus 상품
public class FocusProduct implements Product{
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Mobile) {
            System.out.println("사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
        } else if (device instanceof Pc) {
            System.out.println("사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.");
        }
    }
}

//Top 상품
public class TopProduct implements Product {
    @Override
    public void applyProduct(Device device) {
        if(device instanceof Mobile) {
            System.out.println("사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
        } else if (device instanceof Pc) {
            System.out.println("사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.");
        }
    }
}

public class Client {
    public static void main(String[] args) {
        //상품 생성
        Product top = new TopProduct();
        Product focus = new FocusProduct();

        //PC의 경우
        Device computer = new Pc();
        top.applyProduct(computer);
        focus.applyProduct(computer);
        System.out.println("=============");

        //모바일의 경우
        Device phone = new Mobile();
        top.applyProduct(phone);
        focus.applyProduct(phone);
    }
}

결과 : 
사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.
사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.
=============
사람인앱 지역/직종 TOP 영역에 공고를 노출한다.
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
```

위 코드를 `Pad`하나 추가한 것만으로도 `Product`를 상속한 모든 상품의 기존 코드가 변경된 것을 확인할 수 있으며 이는 `OCP(Open-Clossed Principle)`에 위배되는 것이다. 만약 상품 개수가 더 늘어난다면 수정이 더욱더 쉽지 않을 것이다.

이제 이것을 방문자 패턴을 이용해 개선해보자.

### 적용후 코드

```java
//Element
public interface Product {
    void accept(Device device);
}

//Concrete Element
//Top 상품
public class TopProduct implements Product {
    @Override
    public void accept(Device device) {
        device.applyProduct(this);
    }
}

//Concrete Element
//Focus 상품
public class FocusProduct implements Product{
    @Override
    public void accept(Device device) {
        device.applyProduct(this);
    }
}

//Visitor
public interface Device {
    void applyProduct(TopProduct topProduct);
    void applyProduct(FocusProduct focusProduct);
}

//Concrete Visitor
public class Pc implements Device {
    @Override
    public void applyProduct(TopProduct topProduct) {
        System.out.println("사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(FocusProduct focusProduct) {
        System.out.println("사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.");
    }
}

//Concrete Visitor
public class Mobile implements Device {
    @Override
    public void applyProduct(TopProduct topProduct) {
        System.out.println("사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(FocusProduct focusProduct) {
        System.out.println("사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
    }
}

//Concrete Visitor
public class Pad implements Device{
    @Override
    public void applyProduct(TopProduct topProduct) {
        System.out.println("패드용 사람인앱 지역/직종 TOP 영역에 공고를 노출한다.");
    }

    @Override
    public void applyProduct(FocusProduct focusProduct) {
        System.out.println("패드용 사람인앱 지역/직종 Focus 영역에 공고를 노출한다.");
    }
}

public class Client {
    public static void main(String[] args) {
        Product top = new TopProduct();
        Product focus = new FocusProduct();

        Device computer = new Pc();
        top.accept(computer);
        focus.accept(computer);
        System.out.println("=============");

        Device phone = new Mobile();
        top.accept(phone);
        focus.accept(phone);
        System.out.println("=============");

        Device pad  = new Pad();
        top.accept(pad);
        focus.accept(pad);
    }
}

결과 :
사람인 웹페이지 지역/직종 TOP 영역에 공고를 노출한다.
사람인 웹페이지 지역/직종 Focus 영역에 공고를 노출한다.
=============
사람인앱 지역/직종 TOP 영역에 공고를 노출한다.
사람인앱 지역/직종 Focus 영역에 공고를 노출한다.
=============
패드용 사람인앱 지역/직종 TOP 영역에 공고를 노출한다.
패드용 사람인앱 지역/직종 Focus 영역에 공고를 노출한다.
```


위 코드를 보면 `Element(Product)`에는 `Visitor(Device)`를 인자로 받는 `accept()`가 정의되어있다.

그리고 각 상품에서 오버라이딩한 것을 보면 `Device`의`applyProduct()`를 호출하면서 `this(Product Instance)`인자를 보내게 된다. 

`Visitor`에는 `Device`를 상속한 클래스 인자로 오버로딩한 메소드가 정의됨으로써 각 `Device instance`별로 처리되는 구조가 된다.

프로세스를 간단히하면 다음과 같다.

> `Main` → `Element.accept(Visitor)` → `Visitor.applyProduct(Element)` → `Element Instance에 따른 오버로딩 처리`
> 

이렇게 코드를 짤 경우 이제 만약 다른 `Device`가 추가된다 하더라도 기존의 코드는 전혀 변경되지 않고 `Device`를 상속한 클래스 하나만 정의해주면 된다. 즉 OCP를 만족하게 된다.
