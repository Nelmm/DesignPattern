# 브릿지 패턴

# 정의

> `추상`적인 것과 `구체`적인 것을 **분리**하여 연결하는 패턴
> 

![Untitled](https://user-images.githubusercontent.com/32676275/156380633-f3ad429f-ecb7-4fb2-ab05-f0e2ef3da927.png)

`Abstraction` : 기능 계층의 최상위 클래스이며 추상 인터페이스

`RefinedAbstraction` : 기능 계층에서 새로운 부분을 확장할 클래스

`Implementor` : `Abstraction`의 기능을 구현하기 위한 인터페이스 정의

`ConcreteImplementor` : 실제 기능 구현 클래스

---

# 예제

> 적용전 코드
> 

```java
//외부 APi
public class KakaoMapApi{
    public void drawMap(int x,int y){
        System.out.println("카카오는 " + x + "와 " + y + "로 지도를 그립니다.");
    }
}

public class NaverMapApi {
    Integer x;
    Integer y;

    public void setX(int x) {
        this.x = x;
    }

    public void setY(int y) {
        this.y = y;
    }

    public void sketchMap(){
        if(x == null || y == null) throw new NullPointerException("x와 y는 null일 수 없습니다.");
        System.out.println("네이버는 " + x + "와 " + y + "로 지도를 그립니다.");
    }
}

//Kakao 구현체
public abstract class KakaoMapPage {
    protected final int x;
    protected final int y;
    protected final KakaoMapApi mapApi = new KakaoMapApi();

    public KakaoMapPage(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public abstract void drawPage();
}

public class MyKakaoMapPage extends MapPage {
    public MyMapPage(int x, int y) {
        super(x,y);
    }

    public void drawPage(){
        System.out.println("나의 페이지에 지도 그리기");
        Systme.out.println("카카오 맵 이용");
        mapApi.drawMap(x,y);
    }
}

public abstract class NaverMapPage {
    protected final int x;
    protected final int y;
    protected final NaverMapApi mapApi = new NaverMapApi();

    public NaverMapPage(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public abstract void drawPage();
}

public class MyNaverMapPage extends MapPage {
    public MyMapPage(int x, int y) {
        super(x,y);
    }

    public void drawPage(){
        System.out.println("나의 페이지에 지도 그리기");
        Systme.out.println("카카오 맵 이용");
        mapApi.drawMap(x,y);
    }
}

//API 사용
public static void main(String[] args){
		MyKakaoMapPage kakao = new MyKakaoMapPage();
		MyNaverMapPage naver = new MyNaverMapPage();
}
```

나의 페이지를 그리기 위해 외부 `api`를 사용하는데 이때 `kakaoMapApi`와 `NaverMapApi`가 임의로 저런 형태로 존재한다고 가정을 해보자 이때, 두 `api`는 맵을 그리는 방식도 제공하는 메서드도 사용방법도 다르다

그렇다면 우리가 일반적으로 할 수 있는 생각은 위 코드의 `KakaoMapPage`, `NaverMapPage` 를 각각 구현한 것 처럼 `api`를 구현하는 클래스를 따로 구현하는 것이다. 하지만 이렇게 작성할 경우 `api`가 추가될때마다 새롭게 클래스를 계속해서 구현해줘야하는 번거로움이 생긴다. 그리고 `api`마다 결국은 목적이 비슷하기 때문에 중복된 코드도 계속해서 발생할 가능성이 있다.

여기서 사용할 수 있는 패턴이 `브릿지(Bridge) 패턴`이다.

> 브릿지 패턴 적용 코드
> 

기존 코드에서 `api`를 이용하여 지도를 그리는 **구현** 부분을 별도의 `인터페이스`로 분리하고 `클래스`에서 이 추상적인 `인터페이스`를 사용하도록하여 `종속관계`를 피하고 `결합도`를 낮추어 구현부분에 해당하는 객체의 범용성을 높이도록 설계를 해보자.

```java
//Implementor
public interface MapApi {
    public void drawMap(int x, int y);
}

//ConcreteImplementor
public class MyNaverMapApi implements MapApi{
    private final NaverMapApi mapApi = new NaverMapApi();

    @Override
    public void drawMap(int x, int y) {
        mapApi.setX(x);
        mapApi.setY(y);
        mapApi.sketchMap();
    }
}

//ConcreteImplementor
public class MyKakaoMapApi implements MapApi{
    private final KakaoMapApi mapApi = new KakaoMapApi();

    @Override
    public void drawMap(int x, int y) {
        mapApi.drawMap(x,y);
    }
}

//Abstraction
public abstract class MapPage {
    protected final int x;
    protected final int y;
    protected final MapApi mapApi;

    public MapPage(int x, int y,MapApi mapApi) {
        this.x = x;
        this.y = y;
    }

    public abstract void drawPage();
}

//RefinedAbstraction
public class MyMapPage implements MapPage{
    public MyMapPage(int x, int y,MapApi mapApi) {
        super(x,y,mapApi);
    }

    public void drawPage(){
        System.out.println("나의 페이지에 지도 그리기");
        mapApi.drawMap(x,y);
    }
}

//Client
public class Client {
    public static void main(String[] args) {
        double x = 20.1234;
        double y = 1.35452;
        MyMapPage page1 = new MyMapPage(x,y,new MyNaverMapApi());
        page1.drawPage();   //나의 페이지에 지도 그리기
                            //네이버는 20.1234와 1.35452로 지도를 그립니다.

        MyMapPage page2 = new MyMapPage(x,y,new MyKakaoMapApi());
        page2.drawPage();   //나의 페이지에 지도 그리기
                            //카카오는 20.1234와 1.35452로 지도를 그립니다.
    }
}
```

위의 코드를 보면 최상위 기능 계층의 추상 클래스 `MapPage(Abstraction)` 를 확장한`MyMapPage(RefinedAbstraction)`에서 `Implementor`인 `MapApi`를 분리하여 `ConcreteImplementor`인 `MyNaverMapApi`, `MyKakaoMapApi` 로 확장하여 사용할 수 있게 만들었다. `Client`에서는 사용할 `api(ConcreteImplementor)` 를 선택해서 인자로 보내 기능을 사용할  수 있다.

---

# 장/단점

> 장점
> 
- 추상적인 코드를 구체적인 코드 변경 없이도 독립적으로 확장가능(OCP)
→ 추상 클래스의 `drawPage()`는 그대로 유지하지만 `api` 구현체에 따라 기능이 달라짐
- 단일 책임의 원칙을 지키며 설계 가능
- 런타임에 기능(구현부)를 변경 가능

> 단점
> 
- 계층 구조가 늘어나서 복잡도 증가
→ 하나의 기능에 결국 추상부와 구현부가 나눠지면서 발생

---

# 또 다른 적용

> 시스템 설계
> 

![Untitled](https://user-images.githubusercontent.com/32676275/156380812-9ac634c7-da6b-4cb8-8bff-c38dfeae0e42.png)

> Slf4j
> 

`Logger`: 추상화. 내부적인 메소드는 그대로 사용하면 되며, 실제로 내부에 로그 라이브러리만 갈아 끼우면 된다. (`log4j2`, `log4j`, `logback`와 같이 내부를 갈아 끼우면 되는 방식)

![Untitled](https://user-images.githubusercontent.com/32676275/156380830-68e19ef6-e47f-462c-be2d-52ad9b8088b0.png)

→ 이런식으로 `Slf4J`를 가장 먼저 두면서 ⇒ 다른 `log`라이브러리를 사용하면, 사용성은 유지하되, 다른 새로운 라이브러리로 대체 가능하게 만들 수 있다.

> Java List Interface
> 

![Untitled](https://user-images.githubusercontent.com/32676275/156380859-34d13bdc-8ad5-46a0-a8e6-274ab5cc32c6.png)

![Untitled](https://user-images.githubusercontent.com/32676275/156380875-6f173981-3bbe-43ad-9dbb-3bb1ffe795cd.png)

```java
public void list() {
		List<String> arrayList = new ArrayList<>();
		List<String> linkedList = new LinkedList<>();
}
```

위 처럼 `List<> Interface` 추상 클래스는 그대로 있지만 안의 구현체를 `ArrayList`, `LinkedList`등 무엇을 사용할지 결정
