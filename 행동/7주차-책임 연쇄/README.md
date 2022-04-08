**목차**

- [1. 책임 연쇄 패턴이란](#1-책임-연쇄-패턴이란)
- [2. 코드로 알아보는 책임 연쇄 패턴](#2-코드로-알아보는-책임-연쇄-패턴)
- [3. 장단점](#3-장단점)
    * [3-1. 장점](#3-1-장점)
    * [3-2. 단점](#3-2-단점)
- [4. 다른 패턴과 비교](#4-다른-패턴과-비교)
    * [4-1. 프록시 패턴](#4-1-프록시-패턴)
    * [4-2. 데코레이터 패턴](#4-2-데코레이터-패턴)
    * [4-3. 책임 연쇄 패턴](#4-3-책임-연쇄-패턴)


# 책임 연쇄 패턴```java
@Getter
@Setter
public class IdeaRequest {
private String planner;
private String developer;
private String tester;

}

public class DefaultWorkHandler {
public void handle(IdeaRequest ideaRequest) {
System.out.println("최재우1이 개발을 시작하다");
ideaRequest.setDeveloper("최재우1");
}
}

public class Application {
public static void main(String[] args) {
IdeaRequest ideaRequest = new IdeaRequest();
DefaultWorkHandler handler = new DefaultWorkHandler();
handler.handle(ideaRequest);
}
}

//결과
최재우1이 개발을 시작하다
```

위 처럼 간단히 개발을 진행하는 `DefaultWorkHandler`가 있다해보자

만약 여기에 개발전에 기획을 먼저 처리해야하는 과정이 들어가야한다 해보자.

그렇다면 아래처럼 `DefaultWorkHandler.handle()`에 기획작업을 추가하거나 `DefaultWorkHandler` 를 상속한 `PlanningWorkHandler`를 만들어  기획작업을 처리후 `super()`를 통해 부모 클래스 `DefaultWorkHandler.handle()` 를 처리할 수 있을 것이다.

```java
public class DefaultWorkHandler {
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("최재우2가 기획을 시작하다");
        ideaRequest.setPlanner("최재우2");
        System.out.println("최재우1이 개발을 시작하다");
        ideaRequest.setDeveloper("최재우1");
    }
}
//결과
최재우2이 기획을 시작하다
최재우1이 개발을 시작하다

public class PlanningWorkHandler extends DefaultWorkHandler{
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("최재우2이 기획을 시작하다");
        ideaRequest.setPlanner("최재우2");
        super.handle(ideaRequest);
    }
}

public class Application {
    public static void main(String[] args) {
        IdeaRequest ideaRequest = new IdeaRequest();
        DefaultWorkHandler handler = new PlanningWorkHandler();
        handler.handle(ideaRequest);
    }
}

//결과
최재우2이 기획을 시작하다
최재우1이 개발을 시작하다
```

일단 기존 `DefaultWorkHandler`에 기능을 추가하게되면 `단일 책임 원칙(single responsibility principle)`을 위반하게 된다.

`PlaningWorkHandler`를 구현하는 경우에는 `SRP` 를 만족하지만 클라이언트에서 구체적인 구현체를 변경해줘야한다. 즉 `커플링` 이 높아진다.

만약 기획, 개발후 테스트까지 들어간 `handler`를 하게되면 또 `DefaultWorkHandler`를 상속한 `TesterWorkHandler`를 생성해야하고 만약 기획을 뺀 개발, 테스트만 진행하는 `Handler`를 만들어야할 경우 골치 아파진다.

이제 이러한 문제를 `책임 연쇄 패턴`을 이용해 구현해보자

```java
//처리할 request 클래스
//여기서는 아이디어
@Getter
@Setter
public class IdeaRequest {
    private String planner;
    private String developer;
    private String tester;
}

//Handler 인터페이스
public interface WorkHandler {
    void handle(IdeaRequest ideaRequest);
}

//기획자 Handler
public class PlannerWorkHandler implements WorkHandler{
    private WorkHandler workHandler;

    public PlannerWorkHandler(WorkHandler handler) {
        this.workHandler=handler;
    }
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("기획자가 기획을 한다");
        ideaRequest.setPlanner("최재우2");
        if(this.workHandler != null) {
            this.workHandler.handle(ideaRequest);
        }
    }
}

//개발자 Handler
public class DeveloperWorkHandler implements WorkHandler{
    private WorkHandler workHandler;

    public DeveloperWorkHandler(WorkHandler handler) {
        this.workHandler=handler;
    }
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("개발자가 개발을 한다");
        ideaRequest.setDeveloper("최재우1");
        if(this.workHandler != null && !ideaRequest.getDeveloper().equals("최재우1")) {
            this.workHandler.handle(ideaRequest);
        }
    }
}

//테스터 Handler
public class TesterWorkHandler implements WorkHandler{
    private WorkHandler workHandler;
    public TesterWorkHandler(WorkHandler handler) {
        this.workHandler=handler;
    }
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("테스터가 테스트를 한다");
        ideaRequest.setTester("최재우3");
        if(this.workHandler != null) {
            this.workHandler.handle(ideaRequest);
        }
    }
}

public class Application {
    private WorkHandler workHandler;
    public Application(WorkHandler workHandler) {
        this.workHandler = workHandler;
    }
    public void doWork() {
        IdeaRequest ideaRequest = new IdeaRequest();
        this.workHandler.handle(ideaRequest);
        System.out.println("============");
        printResult(ideaRequest);
    }
    public void printResult(IdeaRequest ideaRequest) {
        System.out.println("기획 작업자 : " + ideaRequest.getPlanner());
        System.out.println("개발 작업자 : " + ideaRequest.getDeveloper());
        System.out.println("테스터 작업자 : " + ideaRequest.getTester());
    }
    public static void main(String[] args) {
        WorkHandler workHandler = new PlannerWorkHandler(new DeveloperWorkHandler(new TesterWorkHandler(null)));
        Application application = new Application(workHandler);
        application.doWork();
    }
}

//결과
```

위 코드를 보면 각 직군(개발, 기획, 테스트)으로 책임을 분리했다.

`Application`에서 보면 `Application`객체가 생성될 때 `Handler`를 정의한 후 생성자로 전달한다. 이렇게 되면 `Application`객체는 생성자로 전달된 `Handler`가 어떻게 체이닝 되어있는지 알 필요 없이 `handle()`기능만 사용하면 된다. 만약 위에서 말했던 테스트 없이 `개발`, `기획`만 `handle()`하고 싶다면 위 코드처럼 로직처럼 특정 개발자가 작업했을 때 테스트 `handle()`을 수행하지 않게 할 수 있다.

---

> 장점
>
- 클라이언트 입장에서 `Handler`를 구체적으로 알 필요 없음 (`디커플링`)
- `SRP`를 만족한다. → 기능을 수정할 때 용이
- `Handler`를 쉽게 추가할 수 있음

> 단점
>
- 코드가 복잡해져 디버깅이 힘듦

> `명령 객체`와 `일련의 처리 객체`를 포함하는 디자인 패턴  
> 요청을 보내는 쪽과 요청을 처리하는 쪽을 분리!

## 1. 책임 연쇄 패턴이란

![Untitled](https://user-images.githubusercontent.com/32676275/148357798-f1430c86-0bb2-4632-9131-59065408ec2e.png)

책임 연쇄 패턴은 클라이언트와 처리하는 객체간의 `디커플링`과 `단일 책임 원칙`을 지키기 위해 체인처럼 객체를 여러개 연결시켜 처리하는 디자인 패턴이라 생각하면 된다.


## 2. 코드로 알아보는 책임 연쇄 패턴

```java
@Getter
@Setter
public class IdeaRequest {
    private String planner;
    private String developer;
    private String tester;
	
}

public class DefaultWorkHandler {
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("최재우1이 개발을 시작하다");
        ideaRequest.setDeveloper("최재우1");
    }
}

public class Application {
    public static void main(String[] args) {
        IdeaRequest ideaRequest = new IdeaRequest();
        DefaultWorkHandler handler = new DefaultWorkHandler();
        handler.handle(ideaRequest);
    }
}

//결과
최재우1이 개발을 시작하다
```

위 처럼 간단히 개발을 진행하는 `DefaultWorkHandler`가 있다.

만약 여기에 개발전에 기획을 먼저 처리해야하는 과정이 들어가야한다 해보자.

그렇다면 아래처럼 `DefaultWorkHandler.handle()`에 기획작업을 추가하거나 `DefaultWorkHandler` 를 상속한 `PlanningWorkHandler`를 만들어 
기획작업을 처리후 `super()`를 통해 부모 클래스 `DefaultWorkHandler.handle()` 를 처리할 수 있을 것이다.

```java
public class DefaultWorkHandler {
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("최재우2가 기획을 시작하다");
        ideaRequest.setPlanner("최재우2");
        System.out.println("최재우1이 개발을 시작하다");
        ideaRequest.setDeveloper("최재우1");
    }
}

// 결과
최재우2이 기획을 시작하다
최재우1이 개발을 시작하다

public class PlanningWorkHandler extends DefaultWorkHandler{
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("최재우2이 기획을 시작하다");
        ideaRequest.setPlanner("최재우2");
        super.handle(ideaRequest);
    }
}

public class Application {
    public static void main(String[] args) {
        IdeaRequest ideaRequest = new IdeaRequest();
        DefaultWorkHandler handler = new PlanningWorkHandler();
        handler.handle(ideaRequest);
    }
}

// 결과
최재우2이 기획을 시작하다
최재우1이 개발을 시작하다
```

일단 기존 `DefaultWorkHandler`에 기능을 추가하게되면 `단일 책임 원칙(single responsibility principle)`을 위반하게 된다.

`PlaningWorkHandler`를 구현하는 경우에는 `SRP` 를 만족하지만 클라이언트에서 구체적인 구현체를 변경해줘야한다. 즉 `커플링` 이 높아진다.

만약 기획, 개발후 테스트까지 들어간 `handler`를 하게되면 또 `DefaultWorkHandler`를 상속한 `TesterWorkHandler`를 생성해야하고 
만약 기획을 뺀 개발, 테스트만 진행하는 `Handler`를 만들어야할 경우 골치 아파진다.

이제 이러한 문제를 `책임 연쇄 패턴`을 이용해 구현해보자.

```java
// 처리할 request 클래스
// 여기서는 아이디어
@Getter
@Setter
public class IdeaRequest {
    private String planner;
    private String developer;
    private String tester;
}

// Handler 인터페이스
public interface WorkHandler {
    void handle(IdeaRequest ideaRequest);
}

// 기획자 Handler
public class PlannerWorkHandler implements WorkHandler{
    private WorkHandler workHandler;

    public PlannerWorkHandler(WorkHandler handler) {
        this.workHandler=handler;
    }
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("기획자가 기획을 한다");
        ideaRequest.setPlanner("최재우2");
        if(this.workHandler != null) {
            this.workHandler.handle(ideaRequest);
        }
    }
}

// 개발자 Handler
public class DeveloperWorkHandler implements WorkHandler{
    private WorkHandler workHandler;

    public DeveloperWorkHandler(WorkHandler handler) {
        this.workHandler=handler;
    }
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("개발자가 개발을 한다");
        ideaRequest.setDeveloper("최재우1");
        if(this.workHandler != null && !ideaRequest.getDeveloper().equals("최재우1")) {
            this.workHandler.handle(ideaRequest);
        }
    }
}

// 테스터 Handler
public class TesterWorkHandler implements WorkHandler{
    private WorkHandler workHandler;
    public TesterWorkHandler(WorkHandler handler) {
        this.workHandler=handler;
    }
    @Override
    public void handle(IdeaRequest ideaRequest) {
        System.out.println("테스터가 테스트를 한다");
        ideaRequest.setTester("최재우3");
        if(this.workHandler != null) {
            this.workHandler.handle(ideaRequest);
        }
    }
}

public class Application {
    private WorkHandler workHandler;
    public Application(WorkHandler workHandler) {
        this.workHandler = workHandler;
    }
    public void doWork() {
        IdeaRequest ideaRequest = new IdeaRequest();
        this.workHandler.handle(ideaRequest);
        System.out.println("============");
        printResult(ideaRequest);
    }
    public void printResult(IdeaRequest ideaRequest) {
        System.out.println("기획 작업자 : " + ideaRequest.getPlanner());
        System.out.println("개발 작업자 : " + ideaRequest.getDeveloper());
        System.out.println("테스터 작업자 : " + ideaRequest.getTester());
    }
    public static void main(String[] args) {
        WorkHandler workHandler = new PlannerWorkHandler(new DeveloperWorkHandler(new TesterWorkHandler(null)));
        Application application = new Application(workHandler);
        application.doWork();
    }
}

```

위 코드를 보면 각 직군(개발, 기획, 테스트)으로 책임을 분리했다.

`Application`에서 보면 `Application`객체가 생성될 때 `Handler`를 정의한 후 생성자로 전달한다. 
이렇게 되면 `Application`객체는 생성자로 전달된 `Handler`가 어떻게 체이닝 되어있는지 알 필요 없이 `handle()`기능만 사용하면 된다. 

만약 위에서 말했던 테스트 없이 `개발`, `기획`만 `handle()`하고 싶다면 위 코드처럼 로직처럼 특정 개발자가 작업했을 때 
테스트 `handle()`을 수행하지 않게 할 수 있다.

## 3. 장단점

### 3-1. 장점

- 클라이언트 입장에서 `Handler`를 구체적으로 알 필요 없음 (`디커플링`)
- `SRP`를 만족한다. → 기능을 수정할 때 용이
- `Handler`를 쉽게 추가할 수 있음

### 3-2. 단점

- 코드가 복잡해져 디버깅이 힘듦

## 4. 다른 패턴과 비교

`데코레이터 패턴`과 `프록시 패턴`이 `책임 연쇄 패턴`과 매우 유사하기 때문에 차이점을 알아보자.

세 개의 패턴 모두 Wrapper Class와 Real Class의 관계가 `has A` 관계를 가지는 Interface 구조로 되어있다.


### 4-1. 프록시 패턴

![proxy](https://user-images.githubusercontent.com/79291114/148319337-2c017ce9-a774-4471-a71e-d815dfcde372.PNG)

![proxy2](https://user-images.githubusercontent.com/79291114/148319338-730ea72d-5306-4152-a94c-27624ad4417b.PNG)

먼저 프록시 패턴은 위와 같은 구조로 되어있다. 
**Real Object에 대해서 실제로 필요할 때 instance가 생성되고 실제 작업이 진행될 수 있도록 하기 위해 적용되는 패턴**이다.

프록시 패턴에서 Real Object는 Proxy가 감싸고 있으며, Proxy를 통해서 처음 접근 또는 요청이 있을 때 생성이 되어진다. 
Proxy는 Client가 적절한 권한이 있는지 검사 하기도하며 client의 요청을 컨트롤한다.

적절하게 만들어진 Proxy 패턴은 Client가 사용할 때는 해당 Real Object를 만들고 사용하지 않을때는 메모리를 해제하는 등의 작업을 하기도 한다.

예를 들어 JPA에서 DB에 접근할 때 `find()` 메서드를 이용하면 즉시 호출 하지만 `getReference()`를 이용하면 Proxy를 이용하여 
실제 query를 실행할 때만 DB를 통해 접근하기도 한다.


### 4-2. 데코레이터 패턴

![decorator](https://user-images.githubusercontent.com/79291114/148319332-304cad1e-87bb-4b44-b74d-24a4fa891f4c.PNG)

![decorator2](https://user-images.githubusercontent.com/79291114/148319336-b890d5a2-6f5d-44ff-a502-d11fb079464e.PNG)

**데코레이터 패턴은 runtime에 real Object에 기능을 확장하고 싶을 때 사용**한다. 

이렇게 분리된 부가기능을 담은 클래스는 중요한 특징이 있다. 부가기능 외의 나머지 모든 기능은 원래 핵심기능을 가진 클래스로 위임해줘야 한다. 
핵심기능은 부가기능을 가진 클래스의 존재 자체를 모른다. 따라서 부가기능이 핵심기능을 사용하는 구조가 된다.

### 4-3. 책임 연쇄 패턴

![Chain-of-Responsibility](https://user-images.githubusercontent.com/79291114/148319329-06245a91-089b-4d62-b449-ca1dbe8506f7.PNG)
![Chain-of-Responsibility2](https://user-images.githubusercontent.com/79291114/148319331-5b9169ad-82f7-4b6c-9794-b1f9d3c1826b.PNG)

데코레이터와 거의 유사한 구조를 가진다.

하지만 **책임 연쇄 패턴은 서블릿 필터나 스프링 MVC의 인터셉터처럼 어떤 작업을 처리하는 과정중에 여러 책임을 지나도록 
설계하는 패턴**이고, **데코레이터는 기존 코드에 어떤 기능을 부가적으로 추가하고 싶을 때 사용할 수 있는 패턴**이다.

따라서 패턴을 목적으로 구분해야하지 생김새로 구분하게되면 귀에 걸면 귀 걸이 코에 걸면 코걸이라고 생각될 수도 있다.

**Reference**

- [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
- [https://sabarada.tistory.com/60](https://sabarada.tistory.com/60)
- [https://ohgyun.com/313](https://ohgyun.com/313)

