# 브릿지 패턴
구현부분에서 `추상적인 부분`과 `구현 부분`을 분리하여 서로 독립적으로 변경/확장이 가능하게 하는 패턴

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
public interface Page {
    public void drawPage();
}

public class MyPage implements Page {
    private int x;
    private int y;
    private KakaoMapApi mapApi = new KakaoMapApi();

    public Page(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public void drawPage(){
        System.out.println("나의 페이지에 지도 그리기");
        Systme.out.println("카카오 맵 이용");
        mapApi.drawMap(x,y);
    }
}
```
나의 페이지를 그리기 위해 외부 api를 사용하는데 이때 kakaoMapApi와 NaverMapApi가 임의로 저런 형태로 존재한다고 가정을 해보겠습니다. 이때, 두 api는 맵을 그리는 방식도 제공하는 메서드도 사용방법도 다릅니다.

페이지에서 Kakao api를 이용하여 지도를 포함한 페이지를 그리려고 하는데 만일 kakao api가 자주 끊겨 naver api를 이용하려고 한다면 아래와 같이 기존 MyPage를 수정하거나 새로운 Page를 만들어 배포하게 될 것입니다. 또한, 새로운 api가 등장하여 교체하거나 기존 api의 사용방법이 바뀐다면 그때마다 해당 객체는 계속 수정이 일어나게 될 것입니다.
 
```java
public class ModifiedPage implements Page {
    private int x;
    private int y;
    private NaverMapApi mapApi = new NaverMapApi();

    public ModifiedPage(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public void drawPage(){
        System.out.println("나의 페이지에 지도 그리기");
        Systme.out.println("네이버 맵 이용");
        mapApi.setX(x);
        mapApi.setY(y);
        mapApi.sketchMap();
    }
}
```

## 적용후 코드
기존 코드에서 api를 이용하여 지도를 그리는 구현부분을 별도의 인터페이스로 분리하고 클래스에서 추상적인 인터페이스를 사용하도록하여 종속관계를 피하고 런타임환경에서 유연하게 동작할 수 있도록 도와준다.

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
public interface Page {
    public void drawPage();
}

//구현체
public class MyPage implements Page{
    private final int x;
    private final int y;
    private final MapApi mapApi;

    public MyPage(int x, int y,MapApi mapApi) {
        this.x = x;
        this.y = y;
        this.mapApi = mapApi;
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

## 장점
1. 확장에 유연
2. 단일 책임의 원칙을 지키며 설계가 가능

## 적용가능한 경우
- 구현부분이 런타임 바인딩이 필요한 경우
- 여러 객체에서 구현부분을 공유하려고 하는 경우
- 많은 인터페이스와의 결합으로 클래스내 구현부분이 많아지는 경우
   
## 어댑터패턴과의 차이점
어댑터 패턴은 서로 다른 인터페이스를 연결해줌으로써 주로 시스템의 설계가 완료된후에 요구사항이 변경이되었을때 적용하지만 브릿지패턴은 설계를 진행중에 의도적으로 확장을 고려하여 추상/구현부(두 레이어)로 분리하여 이를 연결시켜주는 패턴