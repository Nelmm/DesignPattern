# 프록시, 데코레이터, 책임 연쇄 패턴의 차이점

**책임 연쇄 패턴(Chain of Responsibility)은 요청을 보내는 쪽과 요청을 처리하는 쪽을 분리하는 패턴**입니다.

`데코레이터 패턴`과 `프록시 패턴`이 `책임 연쇄 패턴`과 매우 유사한데, 기존 스터디에서 데코레이터 패턴과 설계 방법이 매우 유사하기 때문에 따로 예시는 들지 않고 세 개의 패턴에 대한 차이점에 대해 알아보겠습니다.



## 프록시 vs 데코레이터 vs 책임 연쇄 패턴

세 개의 패턴 모두 Wrapper Class와 Real Class의 관계가 has A 관계를 가지는 Interface 구조로 되어 있스빈다.



### 프록시 패턴

![proxy](https://user-images.githubusercontent.com/79291114/148319337-2c017ce9-a774-4471-a71e-d815dfcde372.PNG)

![proxy2](https://user-images.githubusercontent.com/79291114/148319338-730ea72d-5306-4152-a94c-27624ad4417b.PNG)

먼저 프록시 패턴은 위와 같은 구조로 되어있습니다. **Real Object에 대해서 실제로 필요할 때 instance가 생성되고 실제 작업이 진행될 수 있도록 하기 위해 적용되는 패턴**입니다.

프록시 패턴에서 Real Object는 Proxy가 감싸고 있으며, Proxy를 통해서 처음 접근 또는 요청이 있을 때 생성이 되어집니다. Proxy는 Client가 적절한 권한이 있는지 검사 하기도하며 client의 요청을 컨트롤합니다.
적절하게 만들어진 Proxy 패턴은 Client가 사용할 때는 해당 Real Object를 만들고 사용하지 않을때는 메모리를 해제하는 등의 작업을 하기도 합니다.

예를 들어 JPA에서 DB에 접근할 때 `find()` 메서드를 이용하면 즉시 호출 하시만 `getReference()`를 이용하면 Proxy를 이용하여 실제 query를 실행할 때만 DB를 통해 접근하기도 합니다.



### 데코레이터 패턴

![decorator](https://user-images.githubusercontent.com/79291114/148319332-304cad1e-87bb-4b44-b74d-24a4fa891f4c.PNG)

![decorator2](https://user-images.githubusercontent.com/79291114/148319336-b890d5a2-6f5d-44ff-a502-d11fb079464e.PNG)

**데코레이터 패턴은 runtime에 real Object에 기능을 확장하고 싶을 때 사용**합니다. 이렇게 분리된 부가기능을 담은 클래스는 중요한 특징이 있습니다. 부가기능 외의 나머지 모든 기능은 원래 핵심기능을 가진 클래스로 위임해줘야 합니다. 핵심기능은 부가기능을 가진 클래스의 존재 자체를 모릅니다. 따라서 부가기능이 핵심기능을 사용하는 구조가 되는 것입니다.



### 책임 연쇄 패턴

![Chain-of-Responsibility](https://user-images.githubusercontent.com/79291114/148319329-06245a91-089b-4d62-b449-ca1dbe8506f7.PNG)
![Chain-of-Responsibility2](https://user-images.githubusercontent.com/79291114/148319331-5b9169ad-82f7-4b6c-9794-b1f9d3c1826b.PNG)

데코레이터와 거의 유사한 구조를 가집니다. 

하지만 **책임 연쇄 패턴은 서블릿 필터나 스프링 MVC의 인터셉터처럼 어떤 작업을 처리하는 과정중에 여러 책임을 지나도록 설계하는 패턴**이고, **데코레이터는 기존 코드에 어떤 기능을 부가적으로 추가하고 싶을 때 사용할 수 있는 패턴**입니다.

따라서 패턴을 목적으로 구분해야하지 생김새로 구분하게되면 귀에 걸면 귀 걸이 코에 걸면 코걸이라고 생각될 수도 있습니다.



---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)

[https://sabarada.tistory.com/60](https://sabarada.tistory.com/60)

[https://ohgyun.com/313](https://ohgyun.com/313)

