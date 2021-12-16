# 브릿지 패턴

구현부분에서 `추상적인 부분`과 `구현 부분`을 분리하여 서로 독립적으로 변경/확장이 가능하게 하는 패턴

<br><br>

## 적용전 코드

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

//구현체
public abstract class MapPage {
    protected final int x;
    protected final int y;
    protected final KakaoMapApi mapApi = new KakaoMapApi();


    public MapPage(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public abstract void drawPage();
}

public class MyMapPage extends MapPage {
    public MyMapPage(int x, int y) {
        super(x,y);
    }

    public void drawPage(){
        System.out.println("나의 페이지에 지도 그리기");
        Systme.out.println("카카오 맵 이용");
        mapApi.drawMap(x,y);
    }
}
```

나의 페이지를 그리기 위해 외부 api를 사용하는데 이때 kakaoMapApi와 NaverMapApi가 임의로 저런 형태로 존재한다고 가정을 해보겠습니다. 이때, 두 api는 맵을 그리는 방식도 제공하는 메서드도 사용방법도 다릅니다.

페이지에서 Kakao api를 이용하여 지도를 포함한 페이지를 그리려고 하는데 만일 kakao api가 자주 끊긴다는 소식을 듣고 naver api도 이용하도록 바꾸려고 한다면 아래와 같이 기존 MyMapPage를 수정하고 이를 상속받는 모든 구현체들의 drawPage()를 수정해주어야 할 것입니다.

또한, 새로운 api가 등장하여 교체하거나 기존 api의 사용방법이 바뀐다면 그때마다 계속 수정이 일어나게 될 것입니다.

```java
public abstract class MapPage {
    protected final int x;
    protected final int y;
    protected final NaverMapApi mapApi = new NaverMapApi();
    protected final KakaoMapApi mapApi = new KakaoMapApi();


    public MapPage(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public abstract void drawPage(String api);
}

public class MyMapPage extends MapPage {
    public MyMapPage(int x, int y) {
        super(x,y);
    }

    public void drawPage(String api){
        if("kakao".equals(api) ){
            System.out.println("나의 페이지에 지도 그리기");
            Systme.out.println("카카오 맵 이용");
            mapApi.drawMap(x,y);
        }else if( "naver".equals(api) ){
            System.out.println("나의 페이지에 지도 그리기");
            Systme.out.println("네이버 맵 이용");
            mapApi.setX(x);
            mapApi.setY(y);
            mapApi.sketchMap();
        }
    }
}
```

<br><br>

## Bridge 패턴 적용후 코드

기존 코드에서 api를 이용하여 지도를 그리는 구현부분을 별도의 인터페이스로 분리하고 클래스에서 이 추상적인 인터페이스를 사용하도록하여 종속관계를 피하고 결합도를 낮추어 구현부분에 해당하는 객체의 범용성을 높이도록 설계를 해보자.

```java
//구현부
public interface MapApi {
    public void drawMap(int x, int y);
}

public class MyNaverMapApi implements MapApi{
    private final NaverMapApi mapApi = new NaverMapApi();

    @Override
    public void drawMap(int x, int y) {
        mapApi.setX(x);
        mapApi.setY(y);
        mapApi.sketchMap();
    }
}

public class MyKakaoMapApi implements MapApi{
    private final KakaoMapApi mapApi = new KakaoMapApi();

    @Override
    public void drawMap(int x, int y) {
        mapApi.drawMap(x,y);
    }
}

//추상층
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

//구현체
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
        MyPage page1 = new MyPage(x,y,new MyNaverMapApi());
        page1.drawPage();   //나의 페이지에 지도 그리기
                            //네이버는 20.1234와 1.35452로 지도를 그립니다.

        MyPage page2 = new MyPage(x,y,new MyKakaoMapApi());
        page2.drawPage();   //나의 페이지에 지도 그리기
                            //카카오는 20.1234와 1.35452로 지도를 그립니다.
    }
}
```
기존에는 외부 api에 따라 map을 그리는 방식이 MapPage에 종속되어 외부객체에서는 접근이 불가능했다면, 브릿지패턴을 통해 설계를 진행한다면 MapPage뿐만이 아닌 다른 객체에서도 외부 api에 맞게 map을 그리는 메서드에 접근이 가능해 얼마든지 결합을 할 수 있다.

<br><br>


## 장점

1. 확장에 유연
2. 단일 책임의 원칙을 지키며 설계가 가능
3. 런타임에 기능(구현부)를 변경이 가능

<br><br>

## 적용가능한 경우

- 여러 객체에서 구현부분을 공유하려고 하는 경우
- 많은 인터페이스와의 결합으로 클래스내 구현부분이 많아지는 경우

Bridge 패턴은 구조설계에 속하는 패턴으로 코드를 통해서만 해당 패턴을 사용할 수 있는 것이 아니라 아래 사진처럼 시스템을 설계하는데 있어서도 볼 수 있다.

![bridge](/구조/4주차-브릿지/image/bridge.jpg)

<br><br>

## 어댑터패턴과의 차이점
객체를 입력으로 받아 해당 객체와 연관된 새로운 클래스나 인터페이스를 참조한다는 점에서는 동일

어댑터 패턴은 서로 다른 인터페이스를 연결해줌으로써 시스템의 설계가 완료된후에 적용할 수 있지만 브릿지패턴은 설계를 진행중에 의도적으로 확장을 고려하여 추상/구현부(두 레이어)로 분리하여 이를 연결시켜주는 패턴

한마디로 말하면 사용 목적이 다른데 어댑터 패턴의 서로 다른 인터페이스를 연결해주는 것이 목적이고, 브릿지 패턴은 구현부분을 별도로 분리하여 객체들의 범용성을 증가시키기고 계층간의 결합도를 낮추는것이 목적