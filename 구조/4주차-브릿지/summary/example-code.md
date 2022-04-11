# 브릿지 패턴 예제

예제 코드 요구사항

- 기존에 채용 공고 페이지에 카카오맵 API를 이용한 회사 위치가 표현되고 있음
- 하지만 카카오맵 API는 매일 18:00~20:00에 멈춤 현상이 있음
- 대안으로 네이버 지도 API를 이용한 회사 위치 표현이 필요함

아래 코드는 기존 카카오맵 API만 적용됐을 때 코드라 가정하겠습니다.

```java
/**
 * 카카오맵 API
 */
public class KakaoMapAPI {
    public void drawMap(String address) {
        System.out.printf("카카오 맵에 주소 %s를 표현한다.\n", address);
    }
}

/**
 * 채용 공고 페이지 클래스
 */
public class RecruitPage {
    //회사 주소
    private String address;
    //카카오맵 API
    private final KakaoMapAPI kakaoMapAPI = new KakaoMapAPI();

    public RecruitPage(String address) {
        this.address = address;
    }

    //카카오맵에 표현
    public void drawMap() {
        kakaoMapAPI.drawMap(address);
    }
}

public class Main {
    public static void main(String[] args) {
        RecruitPage recruitPage = new RecruitPage("서울시 구로구 디지털로 34길 43");
        //지도에 표현
				recruitPage.drawMap();
    }
}

결과
카카오 맵에 주소 서울시 구로구 디지털로 34길 43를 표현한다.
```

여기에 네이버맵 API를 적용한다 가정하고 일반적인 방법으로 구현해보면 다음처럼 구현할 수 있습니다.

```java
/**
 * 카카오맵 API
 */
public class KakaoMapAPI {
		//카카오맵에 표현
    public void drawMap(String address) {
        System.out.printf("카카오 맵에 주소 %s를 표현한다.\n", address);
    }
}

/**
 * 네이버 지도 API
 */
public class NaverMapAPI {
    private String address;

    //주소 저장
    public void setAddress(String address) {
        this.address = address;
    }

    //네이버 지도에 표현
    public void draw() {
        System.out.printf("네이버 지도에 주소 %s를 표현합니다.\n", this.address);
    }
}

/**
 * 채용 공고 페이지 클래스
 */
public class RecruitPage {
    //회사 주소
    private String address;
    //카카오맵 API
    private final KakaoMapAPI kakaoMapAPI = new KakaoMapAPI();
    //네이버 지도 API
    private final NaverMapAPI naverMapAPI = new NaverMapAPI();
    
		public RecruitPage(String address) {
        //주소 저장
				this.address = address;
    }

    //인자에 따라 카카오맵, 네이버 지도를 분기처리
    public void drawCompanyMap(String api) {
        if(api.equals("naver")) {
            naverMapAPI.setAddress(address);
            naverMapAPI.draw();
        } else if(api.equals("kakao")) {
            kakaoMapAPI.drawMap(address);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        RecruitPage recruitPage = new RecruitPage("서울시 구로구 디지털로 34길 43");
        //카카오맵 정상 작동
        recruitPage.drawCompanyMap("kakao");
        //카카오맵 멈춤 -> 네이버 지도 호출
        recruitPage.drawCompanyMap("naver");
    }
}

결과
카카오 맵에 주소 서울시 구로구 디지털로 34길 43를 표현한다.
네이버 지도에 주소 서울시 구로구 디지털로 34길 43를 표현합니다.
```

위 코드를 보면 기존 `RecruitPage` 클래스에 `NaverMapAPI` 클래스 객체를 새로 만든 것을 확인할 수 있다. 그리고 이로 인해 기존 코드 `drawMap()`가 변경됐다. 그렇다면 이후에 또 다른 API를 추가할 때마다 기존 코드가 변경되는 상황이 발생한다.

즉 `OCP(Open/Closed Principle)`에 위배되는 상황이다.

이제 브릿지 패턴을 적용해보자. 아래는 예제 코드의 다이어그램이다.

![Untitled](https://user-images.githubusercontent.com/32676275/156580823-32e4d2e9-fb7a-4e0f-88b3-17f5942fce0e.png)

```java
/**
 * 카카오맵 API
 */
public class KakaoMapAPI {
		//맵을 그리는 API 기능
    public void drawMap(String address) {
        System.out.printf("카카오 맵에 주소 %s를 표현한다.\n", address);
    }
}

/**
 * 네이버 지도 API
 */
public class NaverMapAPI {
		//주소
    private String address;

    //주소 저장
    public void setAddress(String address) {
        this.address = address;
    }

    //맵을 그리는 API 기능
    public void draw() {
        System.out.printf("네이버 지도에 주소 %s를 표현합니다.\n", this.address);
    }
}

/**
 * 채용 공고에 보여줄 맵을 구현할 인터페이스
 * ** Implementor **
 */
public interface Map {
		//맵을 그리는 기능을 하는 메소드
    public void drawMap(String address);
}

/**
 * 카카오 맵 실제 구현부
 * ** ConcreteImplementor **
 */
public class KakaoMap implements Map{
		//카카오맵 API를 사용
    private final KakaoMapAPI kakaoMapAPI = new KakaoMapAPI();

		//맵을 그리는 기능
    @Override
    public void drawMap(String address) {
				//카카오맵 사용 방법대로 drawMap() 호출
        kakaoMapAPI.drawMap(address);
    }
}

/**
 * 네이버 지도 실제 구현체
 * ** ConcreteImplementor **
 */
public class NaverMap implements Map{
		//네이버 지도 API를 사용
    private final NaverMapAPI naverMapAPI = new NaverMapAPI();
		
		//맵을 그리는 기능
    @Override
    public void drawMap(String address) {
				//네이버 지도 사용 방법대로 setter() 다음 draw() 호출
        naverMapAPI.setAddress(address);
        naverMapAPI.draw();
    }
}

/**
 * 지도 API를 사용할 페이지 추상 클래스
 * ** Abstraction **
 */
public abstract class Page {
    protected String address;
    protected Map map;    //Map(* Implementor *) 객체를 가지고 있음

    public Page(String address, Map map) {
        this.address = address;
        this.map = map;
    }

    //기업 위치를 맵에 나타낼 메소드
    public abstract void drawCompanyMap();
}

/**
 * 채용 공고 페이지 클래스
 * ** RefinedAbstraction **
 */
public class RecruitPage extends Page {

    //생성자
    public RecruitPage(String address, Map map) {
        super(address, map);
    }

		//기업 위치를 맵에 나타낼 메소드
    @Override
    public void drawCompanyMap() {
        //각 Map 구현체의 메소드를 수행
        this.map.drawMap(this.address);
    }
}

//메인 클래스
public class Main {
    public static void main(String[] args) {
        String address = "서울시 구로구 디지털로 34길 43";
        KakaoMap kakaoMap = new KakaoMap();
        NaverMap naverMap = new NaverMap();

        //사용할 맵 API를 외부에서 주입해서 사용
        Page recruitPage1 = new RecruitPage(address, kakaoMap);
        recruitPage1.drawCompanyMap();

        Page recruitPage2 = new RecruitPage(address, naverMap);
        recruitPage2.drawCompanyMap();
				
				/*
				추후 다른 맵API를 사용해도 
				Map구현체(KakaoMap, NaverMap 등)를 만들어 주입해주면 된다
				확장 과정에서 기존 코드가 수정되지 않기 때문에 
				OCP(Open/Closed Principle)를 만족하게 된다
				*/
    }
}
결과
카카오 맵에 주소 서울시 구로구 디지털로 34길 43를 표현한다.
네이버 지도에 주소 서울시 구로구 디지털로 34길 43를 표현합니다.
```