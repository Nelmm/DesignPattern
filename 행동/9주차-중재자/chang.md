# 중재자 패턴

여러 객체들이 소통하는 방법을 캡슐화 하는 패턴 → 여러 컴포넌트간의 결합도를 중재자를 통해 낮출 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrkCOt%2FbtqwA3d73qA%2FWzBqVmAnHxKxgR3W3T6Zc0%2Fimg.png)

모든 사람에게 일일히 물어보는 상황을 줄이기 위해서 **중간에 누가 한번에 알려주는 역할**만 하게끔 하는 패턴

중재자쪽에서 모든 의존성을 가지고, 각 파트별로 중재자가 객체화 되고, 그대로 각 파트별로 중재자에다가 메소드를 전달하는 방식.

현실 세계에서는 항공기 중재를 관제탑에서 관리하는 것과 비슷한데, 모든 비행기가 자기 주변의 비행기를 교신하면서 이동한다면, 복잡하고 어떻게 이동해야할지 많이 혼란스럽겠지만. 굳이 그럴 필요 없이 중재자인 관제탑에서 모든 책임을 지고 각 비행기에 명령을 할당하면, 훨씬 편하게 이동할 수 있을 것이다.



![](https://refactoring.guru/images/patterns/diagrams/mediator/problem1-en.png?id=86f99055b3e60bb8834d)

예시에서 보이는 것처럼 각 컴포넌트간의 의존성이 섞이는 경우가 존재하는데, 이런 경우를 방지하기 위해서는

![](https://refactoring.guru/images/patterns/diagrams/mediator/solution1-en.png?id=dd991a5b7830de8d43f8)

아예 모든 의존성을 Dialog에서 가지고 각 패턴마다 어떤 상황에 따라서 특정 행동을 할 수 있게끔 각 컴포넌트별로 이벤트를 전달하거나 값을 전달하는 방식으로 처리하면 훨씬 관리할 것들이 줄어서 편리하다. 



중재자 패턴은 더 나아가서 객체 생성, 소멸등의 책임을 부여하면 factory, facade 패턴이랑 비슷하게 작동할 수 있다. 

- ***Facade***  패턴은 객체의 서브시스템 인터페이스를 단순화하는 방향으로 정의. 그러나 새로운 기능을 넣진 않음. 서브시스템 자체가 파사드를 인식할순 없고, 하위 시스템 자체는 파사드를 몰라도, 서브 시스템 자체의 객체끼리 직접 통신 할 수 있음. 
- ***Mediator*** 구성요소간에 통신을 중재자가 다 몰아서 받고, 컴폰넌트끼리는 통신 하지 않음.

그리고 한쪽에 모든 의존을 몰아 줄 수 있기때문에, Controller나 God 객체를 만드는 것을 주의해서 구현하자.

 

## 장점 / 단점

### 장점

컴포넌트 코드를 변경하지 않고 새로운 중재자를 만들어 사용할 수 있다.

각각의 컴포넌트 코드를 보다 간결히 유지시키기 쉬움. 



### 단점

중재자 역할을 하는 클래스와 복잡도 결합도 상승

## 

## 어디서 사용되는가?

JAVA

- ExecutorService
- Executor

Spring

- DispatcherServlet

→ 서로 간의 연결관계가 없는데, 서로를 연관지어서 관리하고 요청을 받아서 처리할 수 있는 패턴

HandlerMapping, HandlerAdapter처럼, 서로간의 의존성이 떨어져서 코드 작성이 유리해지는 방향으로 좋아졌다.

mapping을 통해서 adaptor를 찾고 ... 복잡한 방향으로 갈 수도 있었으나,  두 개를 엮어서 갈 수 있었겠지만, 그런 의존성들을 DispatcherServlet으로 몰아서 의존성을 다 한쪽으로 주고, 각 컴포넌트들은 어렵지 않게 가져가는 방향으로 설계되었다고 봐도 좋아보인다.



출처:

[Mediator](https://refactoring.guru/design-patterns/mediator)