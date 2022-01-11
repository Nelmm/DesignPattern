# 책임 연쇄 패턴

> `명령 객체`와 `일련의 처리 객체`를 포함하는 디자인 패턴
>

![Untitled](https://user-images.githubusercontent.com/32676275/148357798-f1430c86-0bb2-4632-9131-59065408ec2e.png)

책임 연쇄 패턴은 클라이언트와 처리하는 객체간의 `디커플링`과 `단일 책임 원칙`을 지키기 위해 체인처럼 객체를 여러개 연결시켜 처리하는 디자인 패턴이라 생각하면 된다.

예시를 보자.

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