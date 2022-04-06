## 1. 컴포짓 패턴(Composite Pattern)이란?

객체들의 관계를 트리 구조로 구성하여 `부분-전체 계층`을 표현하는 패턴으로, 사용자가 `단일 객체`와 `복합 객체` 모두 동일한 객체로 인식할 수 있는 구조로 이루어 진다.

![composite](https://user-images.githubusercontent.com/79291114/161786374-843c02e2-ff2d-469f-920e-8fdf77379483.PNG)

위의 다이어그램의 각각의 역할은 다음과 같다.

- `Component` : 모든 객체(`부분`, `전체`)의 추상 인터페이스
    - 공통적인 기능을 선언
- `Leaf` : `Component` 인터페이스의 `단일 객체 구현체`
- `Composite` : `Component` 인터페이스의 `전체 관리 객체 구현체`
    - `Leaf`와 똑같은 `Component`의 구현체이지만 `Composite`은 `Leaf`들을 관리하는 기능(add, remove)이 있다.

`컴포짓 패턴`은  전체(`composite`)-부분(`Leaf`) 관계(Ex: Directory-File)를 갖는 객체들 사이의 관계 정의할 때 사용
:arrow_right: 즉 클라이언트에서 `composite` 구현체를 이용해 `Leaf`를 관리하는 것인데 이것을 클라이언트는 `Component`라는 추상 인터페이스로만 기능을 사용하면 된다.





## 2. 코드로 알아보는 컴포짓 패턴

### 2-1. 요구사항
IT전략팀의 이창섭 팀원님이 개발하신 `mohani (mohani.saraminhr.co.kr/board)` 프로젝트를 컴포짓 패턴을 이용하여 예시를 들어본다.

해당 서비스를 이용해보면 가장 먼저보이는 기능은 다음과 같다.

1. 팀단위 통합적인 이슈 시작 ~ 종료 날짜를 보여주는 기능
2. 각 멤버 별 통합적인 이슈 시작 ~ 종료 날짜 보여주는 기능
3. 각각의 이슈의 세부사항을 보여주는 기능

이슈정보라는 단일 객체들의 집합을 가지고 있는 멤버와 멤버들을 가지고있는 팀객체에서 `가장 빠른 이슈 시작 / 가장 늦은 이슈 종료 날짜`를 조회하는 기능은 자기가 가지고 있는 객체의 정보들 중에 min/max를 구하는 것으로 처리 방법이 같다.

때문에, 이슈 날짜를 조회하는 행동을 동일한 인터페이스로 분리하여 취급 할 수 있게 됩니다. 이런 경우 `컴포짓 패턴`을 적용할 수 있다.

해당 예제에서는 `가장 빠른 이슈 시작 조회 / 가장 늦은 이슈 종료 날짜 조회` 라는 행위의 집합을 IssueUser라고 명명하고 코드를 한 번 보겠습니다.

```java
//web에 노출할 이슈에 대한 정보를 담은 VO클래스
public class IssueInfo {
    private final String name;
    private final LocalDate startDate;
    private final LocalDate endDate;
    private final String state;
    private final LocalDateTime updatedDt;
    private final String manager;
    private final String link;
    private final String reporter;
    private final String creator;

    public IssueInfo(String name,LocalDate startDate, LocalDate endDate, String state, LocalDateTime updatedDt, String manager, String link, String reporter, String creator) {
        this.name = name;
        this.startDate = startDate;
        this.endDate = endDate;
        this.state = state;
        this.updatedDt = updatedDt;
        this.manager = manager;
        this.link = link;
        this.reporter = reporter;
        this.creator = creator;
    }

    public String getName() {
        return name;
    }

    public LocalDate getStartDate() {
        return startDate;
    }

    public LocalDate getEndDate() {
        return endDate;
    }
}


//Component
//이슈에 대한 정보를 보여주기위한 서비스인터페이스
public interface IssueUser {
    LocalDate getEarliestStartDateInIssues();   //속한 이슈들 중 가장 빠른 이슈시작날짜
    LocalDate getLatestEndDateInIssues();       //속한 이슈들 중 가장 느린 이슈 종료날짜
}

//Composite
//사람인 개발팀, 팀원들의 집합정보
public class SaraminDevelopmentTeam implements IssueUser{
    private final String name = "사람인개발팀";
    private final List<IssueUser> memberList;

    public SaraminDevelopmentTeam(List<IssueUser> memberList) {
        this.memberList = memberList;
    }

    //사람인 개발팀의 가장 빠른 이슈 시작날짜
    @Override
    public LocalDate getEarliestStartDateInIssues() {
        if(this.memberList.isEmpty()){
            return LocalDate.EPOCH;         //팀원이 없다면 1970.01.01 UTC 00:00:00
        }
        return memberList.stream().min(Comparator.comparing(IssueUser::getEarliestStartDateInIssues)).orElseThrow().getEarliestStartDateInIssues();    //팀의 속한 멤버들의 가장 빠른 이슈시작날짜 중 가장 빠른 이슈 시작 날짜를 가진 멤버 find하여 날짜 get
    }

    //사람인 개발팀의 가장 늦은 이슈 종료날짜
    @Override
    public LocalDate getLatestEndDateInIssues() {
        if(this.memberList.isEmpty()){
            return LocalDate.EPOCH;     //팀원이 없다면 1970.01.01 UTC 00:00:00
        }
        return memberList.stream().min(Comparator.comparing(IssueUser::getLatestEndDateInIssues)).orElseThrow().getLatestEndDateInIssues();     //팀의 속한 멤버들의 이슈 종료날짜 중 가장 늦은 이슈 종료 날짜를 가진 멤버 find 하여 날짜 get
    }

    public List<IssueUser> getIssueUserList() {
        return issueUserList;
    }
}

//Leaf
//팀원 한명의 정보, 멤버가 Leaf객체로 Issue정보들의 집합을 가지고 있다.
public class Member implements IssueUser {
    private final String name;
    private final List<IssueInfo> issueInfoList;    //이슈정보들 집합

    public Member(String name, List<IssueInfo> issueInfoList) {
        this.name = name;
        this.issueInfoList = issueInfoList;
    }

    @Override
    public LocalDate getEarliestStartDateInIssues() {
        if(this.issueInfoList.isEmpty()){
            return LocalDate.EPOCH;
        }
        return issueInfoList.stream().min(Comparator.comparing(IssueInfo::getStartDate)).orElseThrow().getStartDate();  //가진 이슈들 중 가장 빠른 시간 get
    }

    @Override
    public LocalDate getLatestEndDateInIssues() {
        if(this.issueInfoList.isEmpty()){
            return LocalDate.EPOCH;
        }
        return issueInfoList.stream().min(Comparator.comparing(IssueInfo::getEndDate)).orElseThrow().getEndDate();      //가진 이슈들 중 가장 늦은 시간 get
    }
}
```

```java
//사용 예제 코드
public class Client {
    public static void main(String[] args) {
        //임의의 멤버와 이슈 생성
        //이 부분의 코드는 db, api 등 에서 조회후 해당 객체로 mapping 시켜 생성함으로써 대체 될 수 있다.
        Member member1 = new Member("고길동", List.of(
                new IssueInfo("ITIN-1554",
                        LocalDate.of(2022,1,30), LocalDate.of(2022,2,22),
                        "진행중", LocalDateTime.of(LocalDate.of(2022,3,31), LocalTime.now()),
                            "관리자1", "https://example.com", "보고자1", "고길동"),
                new IssueInfo("기업정보뷰 추가",
                        LocalDate.of(2021,12,31), LocalDate.of(2022,2,22),
                        "진행중", LocalDateTime.of(LocalDate.of(2022,4,12), LocalTime.now()),
                        "관리자2", "https://example.com", "보고자2", "고길동"))
        );
        Member member2 = new Member("김둘리",List.of(
                new IssueInfo("노쇼 이벤트",
                        LocalDate.of(2021,10,31), LocalDate.of(2022,2,22),
                        "완료", LocalDateTime.of(LocalDate.of(2021,11,1), LocalTime.now()),
                        "관리자1", "https://example.com", "보고자3", "김둘리"),
                new IssueInfo("채용관리형",
                        LocalDate.of(2022,2,28), LocalDate.of(2022,2,22),
                        "진행중", LocalDateTime.of(LocalDate.of(2022,4,28), LocalTime.now()),
                        "관리자2", "https://example.com", "보고자4", "김둘리"))
        );

        //사람인 서비스 개발팀에 위에서 생성한 고길동, 고둘리 멤버 추가
        SaraminDevelopmentTeam saraminDevelopmentTeam = new SaraminDevelopmentTeam(List.of(member1,member2));

        System.out.println(saraminDevelopmentTeam.getEarliestStartDateInIssues());  //사람인 개발팀의 가장 빠른 이슈 시작 날짜 조회
        System.out.println(saraminDevelopmentTeam.getLatestEndDateInIssues());      //사람인 개발팀의 가장 늦은 이슈 종료 날짜 조회
        for(IssueUser member: saraminDevelopmentTeam.getIssueList()){
            System.out.println(member.getEarliestStartDateInIssues());      //사람인 개발팀에 속한 특정 멤버의 가장 빠른 이슈 시작 날짜 조회
            System.out.println(member.getLatestEndDateInIssues());          //사람인 개발팀에 속한 특정 멤버의 가장 늦은 이슈 종료 날짜 조회
        }
    }
}
```





## 3. 컴포짓 패턴의 방식

컴포짓 패턴에서 `Composite` 클래스는 자식들을 관리하기 위한 추가적인 메서드가 필요하다. 이러한 메서드의 설계 방식에 따라 2가지 형태의 방식으로 나눌 수 있다.

![compositeWay](https://user-images.githubusercontent.com/79291114/161997682-a72c850e-ed90-45d7-99c7-a7f30dd4acf8.PNG)

### 3-1. 안정성을 추구하는 방식

안정성을 추구하는 방식은 자식을 다루는 `add(), remove()` 와 같은 메소드들은 오직 `Composite` 만 정의되었다. 그로 인해, **Client는 Leaf와 Composite을 다르게 취급**하고 있다. 하지만 **Client에서 Leaf객체가 자식을 다루는 메소드를 호출할 수 없기 때문에, 타입에 대한 안정성**을 얻게 된다.



### 3-2. 일관성을 추구하는 방식

일관성을 추구하는 방식은 자식을 다루는 메소드들을 `Composite`가 아닌 `Component`에 정의하는 방식이다. 그로 인해, `Client`는 `Leaf`와 `Composite`를 일관되게 취급할 수 있다. 하지만 `Client`는 `Leaf` 객체가 자식을 다루는 메소드를 호출할 수 있기 때문에, 타입의 안정성을 잃게 된다.

위의 예제는 일관성을 추구하는 방식으로 되어있다.





## 4. 장점과 단점

### 4-1. 장점

- 복잡한 트리 구조를 편하게 사용할 수 있다.
  - 클라이언트는 `IssueUser(컴포넌트)`의 `getEarliestStartDateInIssues(), getLatestEndDateInIssues()` 메서드만 사용하면 되기 때문
- 다형성과 재귀를 활용할 수 있다.
  - 하나의 `getEarliestStartDateInIssues(), getLatestEndDateInIssues` 메서드가 구현체 마다 다르게 동작하는 `다형성`과 `Leaf 객체`를 찾기 위해 `DFS`와 같은 형식의 재귀를 활용하게 된다.
- 클라이언트 코드를 변경하지 않고 새로운 구현체를 추가할 수 있습니다.



### 4-2. 단점

- 트리를 만들야 하기 때문에 (공통된 인터페이스를 정의해야 하기 때문에) 지나치게 일반화 해야 하는 경우가 생길 수 있다.
  - 예를 들어, 기한이 무한인 이슈가 있다면, 이 객체는 `getEarliestStartDateInIssues(), getLatestEndDateInIssues()` 가 굳이 필요하지 않지만 **리스트에 넣으려면 IssueUser(컴포넌트)를 상속받아야 하기 때문에 지나친 일반화가 발생하는 경우**라고 할 수 있다.







## 5. 마치며

컴포짓 패턴을 사용함으로써 `OCP(Open-Closed Principle)` 즉, 개방 폐쇄 원칙을 지키면서 프로그래밍을 할 수 있다는 것을 알 수 있다. 하지만 컴포짓 패턴을 적용하다가 억지로 일반화해야하는 경우가 발생한다면, 해당 구조가 컴포짓 패턴으로 구현하는 게 맞는지 다시 한 번 생각해봐야 한다.